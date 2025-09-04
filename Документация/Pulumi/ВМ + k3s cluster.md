import pulumi
import pulumi_proxmoxve as proxmoxve
import pulumi_command as command

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

# === 1. Создание ВМ ===
vm_count = 6
vms = []
ips = []

for i in range(1, vm_count + 1):
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
            vm_id=9000,  # VM-шаблон
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
                password="userpassword123",
                keys=["ssh-rsa AAAAB3...твой ssh ключ..."]
            )
        ),
        opts=pulumi.ResourceOptions(provider=provider)
    )
    vms.append(vm)
    ips.append(ip_address.split("/")[0])

# === 2. Установка k3s ===

# Подключение к master-1
master1_conn = command.remote.ConnectionArgs(
    host=ips[0],
    user="test_user",
    private_key=pulumi.FileAsset("~/.ssh/id_rsa")
)

# Установка k3s на master-1
master1_install = command.remote.Command(
    "master1-install",
    connection=master1_conn,
    create="curl -sfL https://get.k3s.io | sh -s - server --cluster-init"
)

# Получаем токен для подключения нод
get_token = command.remote.Command(
    "get-token",
    connection=master1_conn,
    create="sudo cat /var/lib/rancher/k3s/server/node-token",
    opts=pulumi.ResourceOptions(depends_on=[master1_install])
)

# Установка k3s на master-2
command.remote.Command(
    "master2-install",
    connection=command.remote.ConnectionArgs(
        host=ips[1],
        user="test_user",
        private_key=pulumi.FileAsset("~/.ssh/id_rsa")
    ),
    create=get_token.stdout.apply(
        lambda token: f"curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - server --server https://{ips[0]}:6443"
    )
)

# Установка k3s на воркеры
for i in range(2, vm_count):
    command.remote.Command(
        f"worker-{i+1}-install",
        connection=command.remote.ConnectionArgs(
            host=ips[i],
            user="test_user",
            private_key=pulumi.FileAsset("~/.ssh/id_rsa")
        ),
        create=get_token.stdout.apply(
            lambda token, ip=ips[i]: f"curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - agent --server https://{ips[0]}:6443"
        )
    )

# === 3. Экспорт kubeconfig ===

kubeconfig = command.remote.Command(
    "get-kubeconfig",
    connection=master1_conn,
    create="sudo cat /etc/rancher/k3s/k3s.yaml",
    opts=pulumi.ResourceOptions(depends_on=[master1_install])
)

# Меняем адрес с localhost на внешний IP master-1
kubeconfig_fixed = kubeconfig.stdout.apply(
    lambda content: content.replace("127.0.0.1", ips[0])
)

# Экспорт
pulumi.export("master_ips", ips[:2])
pulumi.export("worker_ips", ips[2:])
pulumi.export("kubeconfig", kubeconfig_fixed)
