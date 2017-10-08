---
date: 2017-09-24T14:51:00-04:00
title: Service Discovery with Consul
---

Light-4j framework supports both Consul and Zookeeper for service discovery but we prefer
Consul due to some distinct features.

Here is an instruction on how to install Consul 0.9.3 on Ubuntu 16.04 LTS Linux.

The following is a list of three servers and each has 32GB memory. We want to install
consul on three servers to form a production cluster.

* test1.lightapi.net
* test2.lightapi.net
* test3.lightapi.net

## Install Consul 

We follow the [official installation instructions](https://www.consul.io/intro/getting-started/install.html)
and learned a lot details from this [article](https://linoxide.com/devops/install-consul-server-ubuntu-16/)

```
sudo apt-get update
sudo apt-get install unzip
cd /usr/local/bin
sudo wget https://releases.hashicorp.com/consul/0.9.3/consul_0.9.3_linux_amd64.zip
sudo unzip consul_0.9.3_linux_amd64.zip
sudo rm consul_0.9.3_linux_amd64.zip
```

## Start Consul Cluster


