---
title: "Kafka 2.0 Installation"
date: 2017-11-18T16:43:03-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Apache Kafka is a distributed streaming platform developed by Apache Software Foundation and written in Java and Scala. Light Platform uses Kafka for messaging broker for event based frameworks (light-eventuate-4j, light-tram-4j and light-saga-4j). In this document, we will show you how to install Kafka 2.0 on Ubuntu 18.04 VMs to form a three nodes cluster. 

### Prerequisites

Three KVM with 32GB memory each and you need to have sudo access to all the VMs. In the following steps, we are going to use names of the hosts as test1, test2 and test3. First let's login to test1 and install everything. 

### Install OpenJDK 8

Update and upgrade packages

```
sudo apt update
sudo apt upgrade
```

Install JDK

```
sudo apt install openjdk-8-jdk -y
```

Verify installation

```
java -version
```

### Install Zookeeper

Apache Kafka uses zookeeper for the electing controller, cluster membership, and topics configuration. Zookeeper s a distributed configuration and synchronization service.

```
sudo apt install zookeeperd -y
```

### Install Kafka

In this step, we will install the Apache Kafka using the binary files that can be downloaded from the Kafka website. We will install and configure apache Kafka and run it as a non-root user.

Add a new user named kafka

```
sudo useradd -d /opt/kafka -s /bin/bash kafka
```

```
sudo passwd kafka
```

Go to /opt and download kafka.

```
cd /opt
sudo wget http://www.apache.org/dist/kafka/2.0.0/kafka_2.11-2.0.0.tgz
```

Create a new kafka folder

```
sudo mkdir -p /opt/kafka
```
