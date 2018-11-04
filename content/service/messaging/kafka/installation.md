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
reviewed: true
---

Apache Kafka is a distributed streaming platform developed by Apache Software Foundation and written in Java and Scala. Light Platform uses Kafka for messaging broker for event-based frameworks (light-eventuate-4j, light-tram-4j, and light-saga-4j). In this document, we will show you how to install Kafka 2.0 on Ubuntu 18.04 VMs to form a three nodes cluster. 

### Prerequisites

Three KVM boxes with 32GB memory each and you need to have `sudo` access to all the VMs. In the following steps, we are going to use the names of the hosts as test1, test2, and test3. First, let's log in to test1 and install everything. 

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

Extract the kafka_*.tar.gz file to the `kafka` directory and change the owner of the directory to the `kafka` user and group.

```

sudo tar -xf kafka_2.11-2.0.0.tgz -C /opt/kafka --strip-components=1
sudo chown -R kafka:kafka /opt/kafka
```

Create another directory under /opt for kafka logs

```
sudo mkdir /opt/kafka-logs
sudo chown -R kafka:kafka /opt/kafka-logs
```

Create a directory in /var for zookeeper data

```
sudo mkdir /var/zookeeper
sudo chown -R kafka:kafka /var/zookeeper
```

Now login to the `kafka` user and edit the zookeeper.properties configuration

```
su - kafka
vi config/zookeeper.properties
```

```
dataDir=/var/zookeeper
 
server.1=38.113.162.51:2888:3888
server.2=38.113.162.52:2888:3888
server.3=38.113.162.53:2888:3888
#add here more servers if you want
initLimit=5
syncLimit=2

```
Now create a myid file under /var/zookeeper and then insert unique id into this file.

On 38.113.162.51
```
echo "1" > /var/zookeeper/myid
```
On 38.113.162.52
```
echo "2" > /var/zookeeper/myid
```
On 38.113.162.53
```
echo "3" > /var/zookeeper/myid
```


Now edit the server.properties configuration.

```
su - kafka
vi config/server.properties
```

Update the following configuration

```
broker.id=0 # for test1 host, increase for test2 and test3
log.dirs=/opt/kafka-logs # directory we created in the previous step
num.partitions=3
offsets.topic.replication.factor=3
advertised.listeners=PLAINTEXT://38.113.162.51:9092
zookeeper.connect=38.113.162.51:2181,38.113.162.52:2181,38.113.162.53:2181
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

### Test Zookeeper

To confirm zookeeper cluster is running, log in to three nodes and issue command like this.

```
echo mntr | nc localhost 2181
```

Two of them will be followers and one of them should be the leader. 

Here is something output from the leader node. You can see that it has two followers. 


```
zk_version	3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 00:39 GMT
zk_avg_latency	0
zk_max_latency	58
zk_min_latency	0
zk_packets_received	4168
zk_packets_sent	4171
zk_num_alive_connections	3
zk_outstanding_requests	0
zk_server_state	leader
zk_znode_count	32
zk_watch_count	15
zk_ephemerals_count	4
zk_approximate_data_size	1459
zk_open_file_descriptor_count	117
zk_max_file_descriptor_count	4096
zk_fsync_threshold_exceed_count	0
zk_followers	2
zk_synced_followers	2
zk_pending_syncs	0
zk_last_proposal_size	32
zk_max_proposal_size	294
zk_min_proposal_size	32

```

### Install Test2 and Test3

Follow the same steps above to get test2 and test3 installed. Remember that certain variables need to be changed per-host basis. 


### Test Kafka

Login to test1

```
su - kafka
cd bin/
```

Create a new topic. 

```
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic SteveTesting
```

And run the 'kafka-console-producer.sh' with the 'SteveTesting' topic.

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic SteveTesting
```

Now login to test2, then login to the 'kafka' user.

```
su - kafka
cd bin/
```

Run 'kafka-console-consumer.sh' for the 'SteveTesting' topic.

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic SteveTesting --from-beginning
```

And when you type any input from the 'kafka-console-producer.sh' shell on test1, you will get the same result on the 'kafka-console-consumer.sh' shell on test2.

Now login to test3

```
su - kafka
cd bin/
```

Run 'kafka-console-consumer.sh' for the 'SteveTesting' topic.

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic SteveTesting --from-beginning
```

You should see the producer input immedately from the beginning.

### Security

We have both Zookeeper and Kafka clusters running on three nodes and they are exposed on the Internet. We need to find a way to secure both. 


For now, we are going to disable the access to Zookeeper and Kafka from the Internet. Let's enable the firewall with `ufw`.

```
sudo ufw status
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow from 38.113.162.51
sudo ufw allow from 38.113.162.52
sudo ufw allow from 38.113.162.53
```

This means that the cluster can only be accessed from these three nodes locally. If we build applications, we need to deploy them to the three nodes or enable firewall for the hosts with the applications individually. 


