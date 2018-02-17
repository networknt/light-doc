---
title: "Kubernetes"
date: 2018-02-15T21:43:52-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tool"
    weight: 100
weight: 100
aliases: []
toc: false
draft: false
---

This is the tutorial that document the process to install Kubernetes 1.9.3 on four Ubuntu Linux
VMs. 

### Install Master

A new VM is just created and it is called sandbox with 2 CPUs, 4GB memory and 30GB hard drive. 
This server is installed Ubuntu 16.04 LTS server. This server will be acted as a master only
and no pods will be deployed on this VM. 

The first step will install docker and you can refer to 

https://medium.com/@Grigorkh/how-to-install-docker-on-ubuntu-16-04-3f509070d29c

Once docker is installed, we need to install Kubernetes repo

```
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Next let's create a kubernetes.list file

```
sudo vi /etc/apt/sources.list.d/kubernetes.list
```

Add one line as below and save it.

```
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

Now let's install Kubernetes

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubernetes-cni
```

With all the packages installed, we need to initialize kubeadm, but first you need to disable
swap. 

```
sudo swapoff -a
```

To make permanent change you need to comment out the swap file in /etc/fstab file

Before init kubeadm we need to find out the ip address of your master server and it can be found
through ifconfig

```
ifconfig
```

Once you found your network ip address you will use it to replace 10.133.15.28 in the following
command line. 

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.133.15.28 --kubernetes-version stable-1.9
```

After the command is completed, it will show you a command to join other node to the master. 

something like the following.

```
sudo kubeadm join --token 582493.ce5e13aa64d9ccb8 10.133.15.28:6443 --discovery-token-ca-cert-hash sha256:fab46528c4a2e0d38bdf60c6ed5042a60a8535ac48443356c06d3f461204af6d
```

Config a user account or just use the current user if you have logged in with non-root user.

The following is an example if you are using root so far and you want to create a new user
called packet. 

```
sudo useradd packet -G sudo -m -s /bin/bash
passwd packet
```

Switch to the new user with sudo su packet if you have created a new user packet. Otherwise,
stay in the current user session.  


```
$ cd $HOME
$ sudo whoami
$ sudo cp /etc/kubernetes/admin.conf $HOME/
$ sudo chown $(id -u):$(id -g) $HOME/admin.conf
$ export KUBECONFIG=$HOME/admin.conf
$ echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc
```

Apply pod network(flannel)

```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

Use the following command to ensure that all services are up and running on master.

```
kubectl get all --namespace=kube-system
```


### Install Worker

After master is installed and all services are running, you can install workers. Here we are going
to install three works. test1, test2 and test3. 

Make sure that all packages are up to date
```
sudo apt-get update
sudo apt-get upgrade
```

Make sure that docker is installed already.

```
sudo apt-get install -y apt-transport-https
```

install key

```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

create kubernetes.list

```
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```

If above command doesn't work, create the file manually with sudo vi 

Once it is done, we need to install Kubernetes. 

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubernetes-cni
```

Turn off swap by comment out the swap file system

```
sudo vi /etc/fstab
```

Turn off the current session.

```
sudo swapoff -a
```

Now join the cluster with the command given by the master. 

```
sudo kubeadm join --token 582493.ce5e13aa64d9ccb8 10.133.15.28:6443 --discovery-token-ca-cert-hash sha256:fab46528c4a2e0d38bdf60c6ed5042a60a8535ac48443356c06d3f461204af6d
```

To ensure that the cluster is up and running, go to master

```
kubectl get nodes
```

And you should have something like below.

```
packet@sandbox:~$ kubectl get nodes
NAME                 STATUS    ROLES     AGE       VERSION
sandbox              Ready     master    1h        v1.9.3
test1                Ready     <none>    19m       v1.9.3
test2                Ready     <none>    1m        v1.9.3
test3                Ready     <none>    49m       v1.9.3
```


### references

https://blog.alexellis.io/kubernetes-in-10-minutes/
https://medium.com/@Grigorkh/install-kubernetes-on-ubuntu-1ac2ef522a36
