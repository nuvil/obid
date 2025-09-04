https://github.com/spa137/first_steps/blob/main/proxmox%20cloudinit2

wget https://cloud-images.ubuntu.com/minimal/releases/jammy/release/ubuntu-22.04-minimal-cloudimg-amd64.img

qm create 9000 --name "ubuntu-2104-cloudinit-template" --memory 2048 --net0 virtio,bridge=vmbr0

mv ubuntu-22.04-minimal-cloudimg-amd64.img ubuntu-22.04-minimal-cloudimg-amd64.qcow2

qm importdisk 9000 ubuntu-22.04-minimal-cloudimg-amd64.qcow2 local-lvm

qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0

qm set 9000 --ide2 local-lvm:cloudinit

qm set 9000 --boot c --bootdisk scsi0

qm set 9000 --serial0 socket --vga serial0

qm template 9000

qm clone 9000 900 --name my-virtual-machine

qm set --sshkey ~/.ssh/id_rsa.pub

qm set 900 --ipconfig0 ip=192.168.88.253/24, gw=192.168.88.1