import pulumi
import pulumi_proxmoxve as proxmoxve
import pulumi_command as command
import os
import subprocess

# --- Pulumi config для Proxmox ---
config = pulumi.Config("proxmox")
endpoint = config.require("endpoint")
username = config.require("username")
password = config.require_secret("password")

provider = proxmoxve.Provider(
    "proxmoxve",
    endpoint=endpoint,
    username=username,
    password=password,
    insecure=True
)

# --- SSH ключи ---
private_key_path = "C:/Users/<WindowsUser>/.ssh/id_ed25519"
public_key_path = "C:/Users/<WindowsUser>/.ssh/id_ed25519.pub"

private_key = open(private_key_path).read()
public_key = open(public_key_path).read()

# --- Создание ВМ ---
vms = []
ips = []

for i in range(1, 7):  # 6 ВМ
    ip_address = f"192.168.0.{252 + i}/24"
    vm = proxmoxve.vm.VirtualMachine(
        resource_name=f"vm-{i}",
        node_name="srv0",
        vm_id=900 + i,
        name=f"k3s-vm-{i}",
        memory=proxmoxve.vm.VirtualMachineMemoryArgs(dedicated=2048),
        cpu=proxmoxve.vm.VirtualMachineCpuArgs(cores=2, sockets=1),
        clone=proxmoxve.vm.VirtualMachineCloneArgs(
            node_name="srv0",
            vm_id=9000,  # шаблон VM
            full=True
        ),
        initialization=proxmoxve.vm.VirtualMachineInitializationArgs(
            type="nocloud",
            datastore_id="local-lvm",
            ip_configs=[
                proxmoxve.vm.VirtualMachineInitializationIpConfigArgs(
                    ipv4=proxmoxve.vm.VirtualMachineInitializationIpConfigIpv4Args(
                        address=ip_address,
                        gateway="192.168.0.1"
                    )
                )
            ],
            user_account=proxmoxve.vm.VirtualMachineInitializationUserAccountArgs(
                username="test_user",
                keys=[public_key]
            )
        ),
        opts=pulumi.ResourceOptions(provider=provider)
    )
    vms.append(vm)
    ips.append(ip_address.split("/")[0])

# --- Разделение на мастера и воркеры ---
master_ips = ips[:2]
worker_ips = ips[2:]

# --- Проверка SSH доступности для всех ВМ ---
ssh_ready = []
for idx, ip in enumerate(ips):
    conn = command.remote.ConnectionArgs(
        host=ip,
        user="test_user",
        private_key=private_key
    )
    ssh_ready.append(
        command.remote.Command(
            f"ssh-ready-{idx+1}",
            connection=conn,
            create="echo ready",
            opts=pulumi.ResourceOptions(depends_on=[vms[idx]])
        )
    )

# --- Установка k3s на мастер 1 ---
master1_conn = command.remote.ConnectionArgs(
    host=master_ips[0],
    user="test_user",
    private_key=private_key
)

install_prereqs_master1 = command.remote.Command(
    "install-prereqs-master1",
    connection=master1_conn,
    create="sudo apt-get update -y && sudo apt-get install -y curl",
    opts=pulumi.ResourceOptions(depends_on=[ssh_ready[0]])
)

install_k3s_master1 = command.remote.Command(
    "install-k3s-master1",
    connection=master1_conn,
    create="curl -sfL https://get.k3s.io | sh -s - server --cluster-init",
    opts=pulumi.ResourceOptions(depends_on=[install_prereqs_master1])
)

# --- Получение токена ---
get_token = command.remote.Command(
    "get-token",
    connection=master1_conn,
    create="sudo cat /var/lib/rancher/k3s/server/node-token",
    opts=pulumi.ResourceOptions(depends_on=[install_k3s_master1])
)

# --- Установка k3s на мастер 2 ---
master2_conn = command.remote.ConnectionArgs(
    host=master_ips[1],
    user="test_user",
    private_key=private_key
)

command.remote.Command(
    "install-k3s-master2",
    connection=master2_conn,
    create=get_token.stdout.apply(
        lambda token: f"sudo apt-get update -y && sudo apt-get install -y curl && curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - server --server https://{master_ips[0]}:6443"
    ),
    opts=pulumi.ResourceOptions(depends_on=[ssh_ready[1]])
)

# --- Установка k3s на воркеры ---
for idx, ip in enumerate(worker_ips):
    worker_conn = command.remote.ConnectionArgs(
        host=ip,
        user="test_user",
        private_key=private_key
    )
    command.remote.Command(
        f"install-k3s-worker-{idx+1}",
        connection=worker_conn,
        create=get_token.stdout.apply(
            lambda token, ip=ip: f"sudo apt-get update -y && sudo apt-get install -y curl && curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - agent --server https://{master_ips[0]}:6443"
        ),
        opts=pulumi.ResourceOptions(depends_on=[ssh_ready[idx+2]])
    )

# --- Получение kubeconfig ---
raw_kubeconfig = command.remote.Command(
    "get-kubeconfig",
    connection=master1_conn,
    create="sudo cat /etc/rancher/k3s/k3s.yaml",
    opts=pulumi.ResourceOptions(depends_on=[install_k3s_master1])
)

def fix_kubeconfig(content: str) -> str:
    fixed = content.replace("127.0.0.1", master_ips[0])
    fixed = fixed.replace("default", "pulumi-k3s")
    fixed = fixed.replace("default@pulumi-k3s", "pulumi-user@pulumi-k3s")
    return fixed

kubeconfig_fixed = raw_kubeconfig.stdout.apply(fix_kubeconfig)

# --- Сохраняем kubeconfig в файл ---
def write_kubeconfig(content: str):
    path = "./k3s-kubeconfig.yaml"
    with open(path, "w") as f:
        f.write(content)
    print(f"Kubeconfig saved to {path}")

    # --- Проверка кластера ---
    try:
        result = subprocess.run(
            ["kubectl", "--kubeconfig", path, "get", "nodes"],
            capture_output=True,
            text=True,
            check=True
        )
        print("Cluster is ready:\n", result.stdout)
    except Exception as e:
        print("⚠️ Не удалось проверить кластер через kubectl:", e)

    return path

kubeconfig_path = kubeconfig_fixed.apply(write_kubeconfig)

# --- Экспортируем данные ---
pulumi.export("master_ips", master_ips)
pulumi.export("worker_ips", worker_ips)
pulumi.export("kubeconfig_path", kubeconfig_path)