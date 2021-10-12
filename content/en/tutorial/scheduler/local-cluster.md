---
title: "Local Cluster"
date: 2021-10-11T21:54:13-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In most situations, the light-scheduler will be running in a cluster mode. For developers, chances are you need to run the local application in cluster mode to test some use cases. To do that, we will give you the option to start three nodes from a docker-compose in the command line. 

Note: we cannot run multiple instances of the light-scheduler in the command line as these instances are not allowed use with the same local store. 


### Confluent Platform

You have to start the Confluent Platform with the docker-compose in kafka-sidecar repository.

### Create Topics

Create light-scheduler and controller-health-check topics with three partitions from the Control Center.

### Docker Compose

In the light-scheduler folder, start the three-node cluster with docker-compose.

```
cd ~/networknt/light-scheduler
docker-compose up
```

### Test

Query node1, 2, 3 and see the empty response. 

```

curl -k https://localhost:8401/schedulers
curl -k https://localhost:8402/schedulers
curl -k https://localhost:8403/schedulers
```

The response should be empty.

```
[]

```

Create a new task definition on node1

```
curl -k --location --request POST 'https://localhost:8401/schedulers' --header 'Content-Type: application/json' --data-raw '{
  "host": "lightapi.net",
  "name": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
  "action": "INSERT",
  "frequency": {
    "timeUnit": "SECONDS",
    "time": 10
  },
  "topic": "controller-health-check",
  "start": 1633813345308,
  "data": {
    "tlsSkipVerify": "true",
    "address": "172.17.0.1",
    "lastExecuteTimestamp": "0",
    "executeInterval": "0",
    "lastFailedTimestamp": "0",
    "protocol": "https",
    "port": "8443",
    "interval": "10000",
    "id": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
    "deregisterCriticalServiceAfter": "120000",
    "healthPath": "/health/",
    "tag": "dev",
    "serviceId": "com.networknt.petstore-3.0.1"
  }
}'

```

Query the get endpoints on node1, 2, 3, and you will see the same response. 

```
curl -k https://localhost:8401/schedulers
curl -k https://localhost:8402/schedulers
curl -k https://localhost:8403/schedulers

```

From the docker-compose terminal, you can see the task is triggered every 10 seconds on node1. 

Let's stop node1 from another terminal and see what happens.

```
cd ~/networknt/light-scheduler
docker-compose stop scheduler-node1
```

You will see that node2 or node3 takes over the task scheduling and triggers the task every 10 seconds. Also, you will see some Kafka streams rebalancing in the log.


Let's restart node1 in the same terminal.

```
docker-compose start scheduler-node1
```

You will see the rebalancing messages again, and chances are the node1 will take back the scheduling task. 


The above demonstrates the high availability and fault tolerance of the light-scheduler.


### Video Tutorial


{{< youtube bwFotRPPu-Y >}}
