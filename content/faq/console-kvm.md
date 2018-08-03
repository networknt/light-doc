---
title: "Connect to Guess Console from KVM host"
date: 2018-08-01T21:11:30-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

While working on KVM virtual machines, I once locked out of the guess VMs due to ufw enabled and I forgot to allow port 22. After my session is timed out, I cannot connect to my VMs through SSH anymore. The following is the steps to gain access to the VMs and then enable SSH to access them. 

### Gain Console Access to VM

Since SSH is blocked by the firewall on the VM, we have to find another way to connect. 

1. Login to the host machine and try "virsh console api". Chances are it is stuck there without login prompt. 
2. We need to enable console access from the VM. As we cannot login to the VM, we must use virt-edit to update the grub.cfg on the VM to allow ttyS0.

```
sudo apt install libguestfs-tools
sudo virt-edit -d portal /boot/grub/grub.cfg
```

Replace all 'quiet' with 'quiet console=ttyS0' and start portal VM for console connect. You need to shutdown the VM before doing that, otherwise there would be an error. 

3. Issue the following command to start the VM and connect to console

```
virsh start portal && virsh console portal
```

It will take a while for the login prompt to show up.


### Update Firewall Rules

Once you login to the console, issue the following command to allow SSH connect to the VM. 

```
sudo ufw allow from any to any port 22 proto tcp
```

### Test the SSH Connection

Start another terminal and connect to the VM with SSH to test the connection. 

### Exit from the Console

To exit from the console of VM, issue Ctrl+]

