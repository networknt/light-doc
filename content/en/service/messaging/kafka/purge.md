---
title: "Purge Portal Kafka on Test Cloud"
date: 2018-11-10T06:29:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

During the development cycle, chances are you want to reset the entire Kafka cluster to the original state like a refreshed installation. Here are the steps to achieve that. 

### Stop Applications

If consumers or producers are connecting to your Kafka cluster, you need to stop them. For Taiji-chain cluster, we need to stop the chain-writer, chain-reader and token-reader on each Kafka node. 

For the light-portal cluster, we need to stop test1, test2, test3 and test4 docker-compose.

```
ssh test1
cd light-chain/light-config-test/test1
docker-compose down
```

repeat the above for test2, test3 and test4.


Stop the schema-registry server on the sandbox. If this server is left running, the old schemas are still cached and saved back to the new Kafka topic. 

```
ssh sandbox
cd networknt/light-docker
docker-compose -f docker-compose-schema-registry.yml down
```

### Stop Kafka

Stop Kafka instances on three nodes. 

```
sudo systemctl stop kafka
```

### Stop Zookeeper

Stop Zookeeper on three nodes

```
sudo systemctl stop zookeeper
```

### Clean up

Let's clearn up Kafka data in /opt/kafka-logs and zookeeper data in /var/zookeeper/version-2 on each node

```
sudo rm -rf /opt/kafka-logs/*
sudo rm -rf /var/zookeeper/version-2/
```


### Restart

Restart Zookeeper on three nodes


```
sudo systemctl start zookeeper
```

Then restart Kafka on three nodes

```
sudo systemctl start kafka
```

### Verify

Check if /var/zookeeper/version-2 is create. 

Check if there are several checkpoint files created in /opt/kafka-logs folder.

Now you can create your topics again. Have fun. 


### Create topics

Right after the Kafka and Zookeeper clusters are up and running, we need to create the topics with the command line before starting the services with docker-compose or kubectl.

Please follow corresponding instructions to create topics. 
