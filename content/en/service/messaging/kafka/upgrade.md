---
title: "Upgrade Kafka Cluster"
date: 2019-06-04T10:42:17-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Note: After upgrade to 2.2.1, we have upgraded the testnet to 2.3.1 recently by following the same process. 

In the previous section, we have written down the steps to [install][] a Kafka cluster on test1/test2/test3 VMs. The installed version is 2.0.0, which was the latest at the time of the installation. 

The current latest version is 2.2.1, and we are going to upgrade our test cluster to the latest version and see if the data file is compatible. 


### Prepare VMs

Before upgrading to the latest Kafka, we first make sure that Ubuntu 18.04 is upgraded to the most recent. Log in to each of the VM and run the following commands. 

```
sudo apt update
sudo apt upgrade
```

### Download Kafka

Kafka was installed in the /opt folder before. Let's download the latest package to the same folder on each server.

```
cd /opt
sudo wget http://mirror.csclub.uwaterloo.ca/apache/kafka/2.2.1/kafka_2.12-2.2.1.tgz
```

For Kafka 2.3.1 version.

```
cd /opt
sudo wget http://mirror.dsrg.utoronto.ca/apache/kafka/2.3.1/kafka_2.12-2.3.1.tgz
```


### Shutdown

First, let's shutdown all the services that are depending on the Kafka. My services are running in docker containers and it can be checked with `docker ps`

Once the docker instances are found, shutdown them with the docker-compose command. 

Once the services are down, we need to shut down the Kafka on three servers. As you have learned, the Kafka is running as Linux service on each server. We need to shut down Kafka first and then Zookeeper second. 


```
sudo systemctl stop kafka
sudo systemctl stop zookeeper
```


### Backup

If there are previous kafka-logs.tar.gz, zookeeper.tar.gz and kafka.bak folder, remove them before the backup. 

Now, let's backup the data for both Kafka and Zookeeper. 

Kafka data is located in /opt/kafka-logs and Zookeeper data is located at /var/zookeeper

```
cd /opt
sudo tar -czvf kafka-logs.tar.gz kafka-logs
cd /var
sudo tar -czvf zookeeper.tar.gz zookeeper
```

Now, let's move the kafka folder to kakfa.bak

```
cd /opt
sudo mv kafka kafka.bak
```

### Unzip

```
cd /opt
sudo mkdir -p /opt/kafka
sudo tar -xf kafka_2.12-2.2.1.tgz -C /opt/kafka --strip-components=1
sudo chown -R kafka:kafka /opt/kafka
```

For version 2.3.1

```
sudo tar -xf kafka_2.12-2.3.1.tgz -C /opt/kafka --strip-components=1
```


### Update config

Edit zookeeper.properties

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

Edit server.properties

```
vi config/server.properties
```

Update the following configuration

```
broker.id=0 # for test1 host, increase for test2 and test3
log.dirs=/opt/kafka-logs # directory we created in the previous step
num.partitions=3
offsets.topic.replication.factor=3
# change IP for test2 and test3
advertised.listeners=PLAINTEXT://38.113.162.51:9092 
zookeeper.connect=38.113.162.51:2181,38.113.162.52:2181,38.113.162.53:2181
```

### Start Zookeeper

To start Zookeeper and Kafka, we need to exit kafka user as it is not in the sudo list. 


```
sudo systemctl start zookeeper
```

After three servers are up and running, check the zookeeper with

```
echo mntr | nc localhost 2181
```

You should see one of the servers has `zk_synced_followers	2` which indicates the master server.


### Start Kafka

```
sudo systemctl start kafka
```

To ensure that Kafka works, issue the following commands.

```
su - kafka
bin/kafka-topics.sh --list --zookeeper localhost:2181
```


At this moment, we have upgraded the Kafka to 2.2.1 and kept the existing data. You may start your services that are depending on the Kafka cluster.


[install]: /service/messaging/kafka/installation/

