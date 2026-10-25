# mq-automation

A learning project about automated provisioning, configuration as code, 
patching and operational validation of IBM MQ on AlmaLinux using Ansible. 
So far it provisions the VM and installs MQ. More to come as I work through.

## Status

Status as of 25.10.2026
- **Done:** VM provisioned, ping works, snapshot, MQ 9.4.0.10 installed
- **Next:** mq_config role to create the queue manager and queues
- **Next action:** un-comment the `mq_config` line in site.yaml and run `ansible-playbook playbooks/site.yaml` look at the error

## Why

This is a model of middleware lifecycle work done by hand. Automating it speeds up the process and reduces mistakes
from manual deploys. 

## Architecture

Laptop running Arch with Ansible provisions a VM with KVM/libvirt of AlmaLinux 9, Installs IBM MQ. 

## Prerequisites

- Arch host with VT-x / AMD-V enabled
- Packages: `qemu-full libvirt virt-install dnsmasq ansible ansible-lint git`
- `libvirtd` running, your user in the `libvirt` group, and                                              
  `export LIBVIRT_DEFAULT_URI=qemu:///system` in your shell profile
- libvirt's `default` network active and set to autostart (`virsh net-list --all`)
- An SSH key at `~/.ssh/id_ed25519`, loaded with `ssh-add`
- sudo rights on the host (the playbook downloads the cloud image to `/home/libvirt/images`)
- The IBM MQ Advanced for Developers tarball                                                             
  (`9.4.0.10-IBM-MQ-Advanced-for-Developers-LinuxX64.tar.gz`), downloaded yourself                       
  and placed in `files/`. It is not in this repo.
- **If Docker runs on the same host:** allow `virbr0` traffic in the `DOCKER-USER` chain,                
  or the VM can't reach the internet (see "What's broke so far")
 

## Quick start

```bash                                                                                                  
git clone git@github.com:Rolyani/mq-automation.git && cd mq-automation
# copy the MQ tarball into files/
ssh-add
ansible-playbook playbooks/provision.yaml -K    # creates the VM and prints its IP
# put that IP in inventory/hosts.yaml as ansible_host
ansible mq_servers -m ping                       # expect pong
virsh -c qemu:///system snapshot-create-as mq01 clean
ansible-playbook playbooks/site.yaml             # installs MQ
ssh almalinux@<ip> sudo /opt/mqm/bin/dspmqver    # expect Version: 9.4.0.10
```                                     

## What's broke so far

### tc htb qdisc: kernel modules didn't match the running kernel after an update

Had this before when starting a Devpod after updating. The kernel had been updated, so the modules on 
disk no longer matched the running kernel. Laptop needs a reboot to load the new ones.

### dnf timed out from the VM

Docker's firewall only allows its own traffic. No forwarding from the VM. It was not returning anything 
from `ping 1.1.1.1`. I had to allow forwarding from the VM and tell Docker to allow traffic from `virbr0`.                  

### Tarball not found

Simple typo of missing `/`. Read the error logs and added it in.

## Licence

IBM MQ is not redistributed here. Download the developer edition from IBM yourself.
