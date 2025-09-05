import pulumi

import pulumi_proxmoxve as proxmoxve

import pulumi_command as command

import os

  
  
  

# download config

config = pulumi.Config("proxmox")

endpoint = config.require("endpoint")

username = config.require("username")

password = config.require_secret("password")  

private_key = open("C:/Users/PavelSurovtsev/.ssh/id_ed25519").read()

  

# cfg = pulumi.Config()

# private_key = cfg.require_secret("sshPrivateKey")

  

# Provider connection to proxmox

provider = proxmoxve.Provider(

    "proxmoxve",

    endpoint=endpoint,

    username=username,

    password=password,

    insecure=True

)

  

vm_count=6

vms = []

ips = []

  

for i in range(1, vm_count + 1):

    ip_address = f"192.168.0.{129 + i}/24"

    vm = proxmoxve.vm.VirtualMachine(

        resource_name=f"vm-{i}",

        node_name="srv0",        

        vm_id=900 + i,              

        name=f"pulumi-vm-{i}",

        agent=proxmoxve.vm.VirtualMachineAgentArgs(

            enabled=False,

            trim=True,

            type="virtio"

        ),

        bios="seabios",

        memory=proxmoxve.vm.VirtualMachineMemoryArgs(

            dedicated=4096

        ),

        cpu=proxmoxve.vm.VirtualMachineCpuArgs(

            cores=2,

            sockets=1

        ),

        clone=proxmoxve.vm.VirtualMachineCloneArgs(

            node_name="srv0",

            vm_id=9000,

            full=True

        ),

  

        disks=[

            proxmoxve.vm.VirtualMachineDiskArgs(

                interface="scsi0",

                datastore_id="local-lvm",

                size=20,              

                file_format="qcow2"

            )

        ],

        cdrom=proxmoxve.vm.VirtualMachineCdromArgs(

            interface="ide2",

            file_id="none"

        ),

        network_devices=[

            proxmoxve.vm.VirtualMachineNetworkDeviceArgs(

                bridge="vmbr0",

                model="virtio"

            )

        ],

        on_boot=True,

        operating_system=proxmoxve.vm.VirtualMachineOperatingSystemArgs(type="other"),

        initialization=proxmoxve.vm.VirtualMachineInitializationArgs(

            type="nocloud",

            datastore_id="local-lvm",

            dns=proxmoxve.vm.VirtualMachineInitializationDnsArgs(

                domain="example.com",

                servers=["1.1.1.1", "8.8.8.8"]

            ),

            ip_configs=[

                proxmoxve.vm.VirtualMachineInitializationIpConfigArgs(

                    ipv4=proxmoxve.vm.VirtualMachineInitializationIpConfigIpv4Args(

                        address=ip_address,

                        gateway="192.168.0.1"

                    )

                )

            ],

            user_account=proxmoxve.vm.VirtualMachineInitializationUserAccountArgs(

                username="user",

                password="password123",

                keys=["ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ7tQumKqFOKH6fhhuf3MAHjIVe+LKpm8MQnH2zVgnh/ proxmox"]

            )

        ),

        opts=pulumi.ResourceOptions(provider=provider)

    )

    vms.append(vm)

    ips.append(ip_address.split("/")[0])

  
  
  

master1_conn = command.remote.ConnectionArgs(

    host=ips[0],

    user="user",

    private_key=private_key

)

  

master1_install = command.remote.Command(

    "master1-install",

    connection=master1_conn,  

    create="curl -sfL https://get.k3s.io | sh -s - server --cluster-init"

    # create="wget -qO- https://get.k3s.io | sh -s - server --cluster-init"

)

get_token = command.remote.Command(

    "get-token",

    connection=master1_conn,

    create="sudo cat /var/lib/rancher/k3s/server/node-token",

    opts=pulumi.ResourceOptions(depends_on=[master1_install])

)

# # Установка k3s на master-2

command.remote.Command(

    "master2-install",

    connection=command.remote.ConnectionArgs(

        host=ips[1],

        user="user",

        private_key=private_key

    ),

    create=get_token.stdout.apply(

        lambda token: f"curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - server --server https://{ips[0]}:6443"

    )

)

  

#

for i in range(2, vm_count):

    command.remote.Command(

        f"worker-{i+1}-install",

        connection=command.remote.ConnectionArgs(

            host=ips[i],

            user="user",

            private_key=private_key

        ),

        create=get_token.stdout.apply(

            lambda token, ip=ips[i]: f"curl -sfL https://get.k3s.io | K3S_TOKEN={token} sh -s - agent --server https://{ips[0]}:6443"

        )

    )

  

# Экспорт kubeconfig

  

kubeconfig = command.remote.Command(

    "get-kubeconfig",

    connection=master1_conn,

    create="sudo cat /etc/rancher/k3s/k3s.yaml",

    opts=pulumi.ResourceOptions(depends_on=[master1_install])

)

  

# # Меняем адрес с localhost на внешний IP master-1

kubeconfig_fixed = kubeconfig.stdout.apply(

    lambda content: content.replace("127.0.0.1", ips[0])

)

  
  

pulumi.export("vm_names", [vm.name for vm in vms])

pulumi.export("vm_ips", ips)

pulumi.export("kubeconfig", kubeconfig_fixed)