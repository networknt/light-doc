---
title: "Service Discovery"
date: 2020-12-04T22:01:07-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The light-controller is a global registry that manages all the live instances of microservices in the system. Also, the health check ensures the registry's information is up to date all the time, which is very valuable for service discovery. 

In a microservices application, before the consumer invokes a service, it can look up in the registry for all the available nodes and pick up one to establish a connection. After that, the same connection can be cached for a while or for a number of requests configured in the client.yml file.

Once the limit is reached, the consumer will do another lookup to pick up another node. This time, it is not necessary to query the controller through API calls. The controller will keep updating the nodes for the service constantly to the consumer through WebSocket, and the local cache is the latest snapshot on the controller.

The above design is more efficient than pulling from the controller for each lookup as the updates are for all the subscribed downstream services. 

We are going to use the same Consul registry and discovery application to demonstrate the discovery, and the source code can be found in the light-example-4j/discovery folder.

### Controller

To allow other services to register to the controller, we need to start the controller first.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-controler.yml up
```
