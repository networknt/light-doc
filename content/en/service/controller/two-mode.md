---
title: "Two Mode"
date: 2021-10-07T09:09:58-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Users can start the controller in two different modes: Demo and Cluster.

### Demo Mode

The service runs in a single instance for demo mode, and all the registration and health check information is in memory. All APIs are working the same as the cluster mode, but they cannot be scaled. 
Once the instance is down, all information is gone, and all services should be restarted to register again. 

There is no Kafka cluster dependency when running in this mode, and it is easy for users to run it on their desktop or laptop with docker for demo and local development. 


### Cluster Mode

For high availability and fault tolerance, we use Kafka as the backend with Kafka Streams to sync the registration and health check info across all nodes. Also, we use light-scheduler to schedule the health check tasks to distributed execute them across all the nodes and scaled with Kafka partitions. 


### Configuration

To switch between the demo mode and cluster mode, you can change the config properties in controller.yml

```
# Cluster mode that uses Kafka as backend for high availability
clusterMode: ${controller.clusterMode:false}

```

By default, the controller will be running in demo mode and it can be changed with externalized values.yml like the following. 

```
controller.clusterMode: true
server.serviceId: com.networknt.controller-1.0.0
server.httpsPort: 8431
```

