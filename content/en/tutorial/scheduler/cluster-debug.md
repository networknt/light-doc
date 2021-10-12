---
title: "Cluster Debug"
date: 2021-10-12T12:28:48-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In the previous step, we have started a three-node cluster and demonstrate high availability and fault tolerance. In this step, we are going to show developers how to debug in cluster mode. 

This tutorial is for advanced developers trying to debug some cluster-related issues locally, and other users don't need to learn it.

To run a cluster in a mixed-mode, we need to start node2 and node3 in docker-compose and node1 in the IDE with config/local as externalized config. 

There are two options: 

### Stop the node1
We can start the docker-compose in the light-scheduler folder and stop node1 with a docker-compose command. I personally prefer this option as I don't need to update the docker-compose.yml file.

### Comment out the node1
We can comment out the docker-compose.yml in light-scheduler and start only node2 and node3 from the docker-compose.

### Setup IDE

As we are using docker-compose.yml from kafka-sidecar repository, we need to connect the IDE instance with config/local values.yml with -Dlight-4j-config-dir=config/local

### First Time

When the instance starts from the IDE the first time, your stream's local store might not be in sync with the Kafka server changelog topics. You might see some errors within your IDE console. If this happens, stop the server and update the config/local/values.yml file to set kafka-streams.cleanUp to true and restart the server again. Once the server starts successfully, you should update the cleanUp flag to false and restart the server. 

### Test and Debug

Follow the normal steps to test and debug. 

### Video Tutorial

{{< youtube a5ajc0l7_Ew >}}

