# Provision Ubuntu VMs with Vagrant-VMware on Apple Silicon (M1/M2)
This repo aims to provision Ubuntu virtual machines (VMs) with [Vagrant](https://developer.hashicorp.com/vagrant) and [VMware Fusion](https://www.vmware.com/products/fusion.html) on Apple Silicon (M1/M2).

### Using the Ubuntu-ARM64 Vagrant Box
Since VMware Fusion only supports running ARM on ARM, the Vagrant Box used in this repo is [`bento/ubuntu-20.04-arm64`](https://app.vagrantup.com/bento/boxes/ubuntu-20.04-arm64) which is a Ubuntu-ARM64 box, instead of x86.

### Prerequisites
- [Vagrant v2.3.4](https://developer.hashicorp.com/vagrant/downloads) as well as [Plugin](https://developer.hashicorp.com/vagrant/docs/providers/vmware/installation) and [Utility](https://developer.hashicorp.com/vagrant/docs/providers/vmware/vagrant-vmware-utility)
- [VMware Fusion Player v13.0 (Free Personal Use License)](https://customerconnect.vmware.com/en/evalcenter?p=fusion-player-personal-13)

### Directory Structure

```
.
├── README.md
├── Vagrantfile
└── scripts
    ├── collect-host-ip.sh
    └── setup-hosts.sh

1 directory, 4 files
```

### Usage

Provision VM(s).

```
cd vagrant-vmware-arm
vagrant status
vagrant up
```
SSH to the VMs.
```
vagrant ssh [hostname]
```
Suspend the VMs.
```
vagrant suspend
```

### Setup k8s 

- [01-setup_controller.md](docs/01-setup_controller.md)
- [02-setup_worker.md](docs/02-setup_worker.md)