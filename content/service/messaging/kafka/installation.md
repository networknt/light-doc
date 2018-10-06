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

Three KVM boxes with 32GB memory each and you need to have sudo access to all the VMs. In the following steps, we are going to use names of the hosts as test1, test2 and test3. First let's login to test1 and install everything. 

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

Extract the kafka_*.tar.gz file to the 'kafka' directory and change the owner of directory to the 'kafka' user and group.

```

sudo tar -xf kafka_2.11-2.0.0.tgz -C /opt/kafka --strip-components=1
sudo chown -R kafka:kafka /opt/kafka
```

Now login to the 'kafka' user and edit the server.properties configuration.

```
su - kafka
vi config/server.properties
```

Paste the following configuration to the end of the line.

```
delete.topic.enable=true
```


### Config Kafka and Zookeeper as Services

Exit the kafka session and go to the '/lib/systemd/system' directory and create a new service file 'zookeeper.service'.


```
exit
cd /lib/systemd/system/
sudo vi zookeeper.service
```

Paste the configuration below.

```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

Save and exit.

Now create the Apache Kafka service file 'kafka.service'.

```
sudo vi kafka.service
```

Paste the config below. 

```
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties'
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

Save and exit.

Reload the systemd manager configuration

```
sudo systemctl daemon-reload
```

Now start the kafka and zookeeper. 

```
sudo systemctl start zookeeper
sudo systemctl enable zookeeper

sudo systemctl start kafka
sudo systemctl enable kafka
```

Zookeeper running under port '2181', and Kafka on port '9092', check it using the netstat command below.

```
netstat -plntu
```

### Test Kafka

```
su - kafka
cd bin/
```

Create a new topic. 

```
./kafka-topics.sh --create --zookeeper localhost:2181 \
--replication-factor 1 --partitions 1 \
--topic SteveTesting
```

And run the 'kafka-console-producer.sh' with the 'SteveTesting' topic.

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic SteveTesting
```

Now open a new terminal and log in to the server, then login to the 'kafka' user.

Run 'kafka-console-consumer.sh' for the 'SteveTesting' topic.

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic SteveTesting --from-beginning
```

And when you type any input from the 'kafka-console-producer.sh' shell, you will get the same result on the 'kafka-console-consumer.sh' shell.


### Install Test2 and Test3

