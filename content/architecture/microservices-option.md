---
title: "Microservices Cross-cutting Concerns Options"
date: 2019-01-10T20:37:30-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 11
weight: 11
aliases: []
toc: false
draft: false
---

There are three options to address cross-cutting concerns when deploying microservices on the cloud. Chassis, Service Mesh, and Hybrid which combines both Chassis and Service Mesh. In the rest of the article, I will list pros and cons of each option and give my recommeadations. 

Before we move to the details, we need to clarify what is the cross-cutting concerns and what type of the cross-cutting concerns we need to address when deploying microservices. 

Cross-cutting concerns are some business logic that sharable across multiple services or all services. As they are common, it is a good idea to extract them outside of each individual service so that developers can focus on the business logic for the specific service he/she is working on. 

They are two types of cross-cutting concerns: Technical and Business.

Technical cross-cutting concerns work regardless of what kind of industries or businesses. For example, logging, tracing, metrics, and security etc. are common for all application. 

Business cross-cutting concerns must be executed within the business context as they are related to the particular line of business. For example, fine-grained authorization must be running within the business context as it needs additional information from the user database, the policy of each organization etc. 

### Chassis

In this approach, all cross-cutting concerns will be addressed in the process as middleware handlers or plugins. 

##### Pros:

* Highest throughput
* Lowest latency 
* Lowest cost
* Max stability as there are less distributed components.
* Simplest deployment with less component and standard configuration management

##### Cons:

* High cost of developing and maintaining the chassis
* All services have to be built on top of the same chassis
* Higher cost of upgrade the chassis as it is part of the service

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

This approach implements some of the cross-cutting concerns in the chassis and some of them in the mesh. 

##### Pros

* You can choose the best location for a specific cross-cutting concern. Some cross-cutting concerns are more suitable to be implemented in the chassis. 

##### Cons

* Lowest throughput
* Highest latency
* Highest cost as you have to build and maintain the chassis and foot the bill for the mesh
* Too many components interact that impact the stability
* Highest deployment and runtime cost

### Conclusion

For most organizations, it is not recommended implementing a chassis to address technical cross-cutting concerns as it takes a lot of effort and unique skills to build one. Also, as these cross-cutting concerns work at the lower level, they need to be continuously maintained to meet the security and industry standard. It is a good idea to choose an open source chassis with the flexibility to plugin your business cross-cutting concerns to make customized platform. The light-platform provides several chassis implementations to support popular protocols like REST, GraphQL, WebSocket, Mobile, SPA and RPC for the synchronous request/response style and Transastional Messaging, Event Sourcing, CQRS, and Saga for the asynchronous event-based style. It is the ideal platform to plug in your business cross-cutting concerns to make it a customized platform for your particular industry. 

If your organization can standardize the platform, then it is the most cost-effective approach. The engineering team can concentrate the effort and won't be distracted from varying heterogeneous platforms. In this case, a chassis-based approach is the best option. 

If your organization is too big and want to support multiple languages for microservices, then you can choose the second option with service mesh on the network level. However, the amount of effort to support an extra language to build services it huge. If your organization has less than 4000 engineers, it is recommended to use only one language. The number is calculated based on the number of engineers and official languages in Google.

The third option that combines chassis and service mesh is not recommended especially you have to build your own chassis. It combines most of the cons from the other two options without much benefit. If you ever going that approach, it is better to pick up an open source chassis to do so. 

In the long run, it would be better to have a hybrid approach supported by an open source platform that integrates with the service mesh natviely. In the light platform, we have several middleware handlers that talks to CA Layer 7 and RedHat 3 Scale gateways. We are also working on the light-portal exposes service mesh API to integrate with Istio mixers. 





