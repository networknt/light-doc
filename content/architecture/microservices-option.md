---
title: "Microservices Cross-cutting Concerns Options"
date: 2019-01-10T20:37:30-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

There are three options to address cross-cutting concerns when deploying microservices on the cloud. Chassis, Service Mesh, and Hybrid which combines both Chassis and Service Mesh.

### Chassis

In this approach, all cross-cutting concerns will be addressed in the process as middleware handlers or plugins. 

##### Pros:

* Highest throughput
* Lowest latency 
* Lowest cost
* Max stability as there are less distributed components.
* Simplest deployment

##### Cons:

* High cost of developing and maintaining the Chassis
* All services have to be built on top of the chassis
* Higher cost of upgrade the service

### Service Mesh

In this approach, the developer just write the service without thinking about any cross-cutting concerns, they will be addressed by the service mesh in other containers. 

##### Pros:

* Support heterogeneous services
* Easy to upgrade without touching the service container.


##### Cons
* Lower throughput
* Higher latency
* Higher cost for the mesh containers
* Too many components to impact the stability
* Moderate deployment complicity


### Hybrid

implement some of the cross-cutting concerns in the chassis and some of them in the mesh. 

##### Pros

* You can choose the best location for a specific cross-cutting concern. Some cross-cutting concerns are more suitable to be implemented in the chassis. 

##### Cons

Lowest throughput
Highest latency
Highest cost as you have to build and maintain the chassis and foot the bill for the mesh
Too many components interact that impact the stability
Highest deployment and runtime cost.

It combines most of the cons from the above options so it is not recommended. 



