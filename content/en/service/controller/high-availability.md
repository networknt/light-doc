---
title: "High Availability"
date: 2021-07-28T22:13:30-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

### Register and deregister

We are using Kafka streams for the high availability implementation. It is recommended that we have 3 or 5 instances running so that workload can be migrated within them if one or two instances are not available. 

It is very easy to sync the register and deregister info between instances. Any node that receives a register request will send a ControllerRegisterEvent to the portal-event topic. Then all nodes will process the event with Kafka streams to update it in memory projection. Any node that receives a deregister request will send a ControllerDeregisterEvent to the portal-event topic for all nodes to sync theirs in memory projection. 

### Health Checks


### Server Info

Instead of retrieving the server info in the register handler, we change the logic to get the server info once UI is trying to access it. This on-demand access can greatly reduce resource usage and reduce the load on the controller instance as most server info won't be accessed very frequently, and the cache is unnecessary. 

### Websocket Subscription


