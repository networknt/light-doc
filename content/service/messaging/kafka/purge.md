---
title: "Purge Kafka Topic"
date: 2018-11-10T06:29:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

During the development cycle, chances are you want to reset the entire Kafka cluster to the original state just like a referesh installation. Here is the steps to archive that. 

### Stop Applications

If there are consumers or producers that are connecting to your Kafka cluster, you need to stop them. For Taiji-chain cluster, we need to stop the chain-writer and chain-reader on each Kafka node. 


```
ssh test1
cd light-chain/light-docker
docker-compose -f docker-compose-test1.yaml down
```

repeat the above for test2 and test3

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

Let's clearn up Kafka data in /opt/kafka-logs on each node

```
cd /opt/kafka-logs
sudo rm -rf *
```

Let's clean up Zookeeper data in /var/zookeeper on each node

```
sudo rm -rf version-2
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



