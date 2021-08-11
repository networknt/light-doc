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

The controller sends health check requests periodically every 10 seconds configurable. To support high availability and fault-tolerant, we use Kafka Streams and State Store to do so.



### Server Info

Instead of retrieving the server info in the register handler, we change the logic to get the server info once UI is trying to access it. This on-demand access can greatly reduce resource usage and reduce the load on the controller instance as most server info won't be accessed very frequently, and the cache is unnecessary. 

### Websocket Subscription

The light controller is using WebSockets to notify all the subscribed clients for the interested service changes. To scale out the cluster for performance and redundancy, we need to consider the following design ideas.

All changes to the services will be pushed to the portal-event topic and stream processed by all the nodes. It will ensure that all nodes have the latest information about the service changes. 

When a client is connected to a WebSocket server, and that server fails over, the client can open a connection through the load balancer to another WebSocket server. The new WebSocket server will ensure a subscription to the latest service instance snapshot for the WebSocket client's data and start piping through changes on the WebSocket when they occur.

One thing to take into consideration when a client reconnects is making the client intelligent enough that it sends through some sort of data synchronization offset, probably in the form of a timestamp, so that the server doesn’t send it all the data again.

If every update to the service instance is timestamped, the clients can easily store the latest timestamp that they received. When the client loses the connection to a particular server, it can just reconnect to your WebSocket cluster (through your load balancer) by passing in the last timestamp that it received and that way the query to the snapshot can be built up so that it’ll only return updates that occur after the client last successfully received updates.
