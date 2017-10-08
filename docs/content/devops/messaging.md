---
date: 2017-09-24T14:50:41-04:00
title: Messaging
---

Light-4j framework support both synchronous request/response style and asynchronous 
message driven style of service to service communication. So a reliable and scalable
message broker is very important in the infrastructure and we have chosen Kafka as
part of our infrastructure. 

For production installation, you need at least three servers to form a cluster. The
following instruction is for Confluent Platform Open Source Edition v3.3

The official installation document can be found [here](https://docs.confluent.io/current/installation.html) 

Here are the three servers and each has 32GB memory.

* test1.lightapi.net
* test2.lightapi.net
* test3.lightapi.net

## Install Oracle JDK 8

We need to install JDK 8 on three servers before installing Confluent. Here is the
[instruction on JDK 8 installation](https://networknt.github.io/light-4j/devops/java/) 


## Install Confluent

There are many different ways to install Confluent and here we are using apt-get to
install it.

For other ways to install it, please refer to [official document](https://docs.confluent.io/current/installation.html#installation-apt)

```
wget -qO - http://packages.confluent.io/deb/3.3/archive.key | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] http://packages.confluent.io/deb/3.3 stable main"

sudo apt-get update && sudo apt-get install confluent-platform-oss-2.11

```

## Start Confluent Cluster