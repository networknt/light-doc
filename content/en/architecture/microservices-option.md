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
reviewed: true
---

There are three options to address cross-cutting concerns when deploying microservices on the cloud: Chassis, Service Mesh, and Hybrid, which combines both Chassis and Service Mesh. In the rest of the article, I will list the pros and cons of each option and give my recommendations. 

Before we move to the details, we need to clarify the  cross-cutting concerns and what type of cross-cutting concern we need to address when deploying microservices.

Cross-cutting concerns are some business logic that is shareable across multiple services or all services. As they are common, it is a good idea to extract them outside of each individual service so that developers can focus on the business logic for the specific service he/she is working on.

They are two types of cross-cutting concerns: Technical and Business.

Technical cross-cutting concerns work regardless of what kind of industries or businesses there are. For example, logging, tracing, metrics, and security, etc. are common for all applications.

Business cross-cutting concerns must be executed within the business context as they are related to the particular line of business. For example, fine-grained authorization must be running within the business context as it needs additional information from the user database, the policy of each organization, etc. 
 

### Chassis

In this approach, all cross-cutting concerns will be addressed in the process as middleware handlers or plugins. Most importantly, these middleware handlers and plugins should be configured transparently by the operation team instead of developers. Some of the frameworks use code or annotation to wire in cross-cutting concerns, and it puts a lot of pressure on the developers. Most developers should only focus on the business logic implementation in a microservices architecture.  

##### Pros:

* Highest throughput
* Lowest latency 
* Lowest cost
* Max stability as there are less distributed components.
* Simplest deployment with less component and standard configuration management

##### Cons:

* Higher cost of upgrading the chassis as it is part of the service
* All services have to be built on top of the same chassis
* Higher cost of upgrade the chassis as it is part of the service

### Service Mesh

In this approach, the developer writes the service without thinking about any cross-cutting concerns; the service mesh in other containers will address them separately. 

##### Pros:

* Supports heterogeneous services
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

For most organizations, it is not recommended to implement a chassis to address technical cross-cutting concerns as it takes a lot of effort and unique skills to build one. Also, as these cross-cutting concerns work at the lower level, they need to be continuously maintained to meet the security and industry standard. It is a good idea to choose an open-source chassis with the flexibility to plugin your business cross-cutting concerns to make a customized platform. The light-platform provides several chassis implementations to support popular protocols like REST, GraphQL, WebSocket, Mobile, SPA and RPC for the synchronous request/response style and Transactional Messaging, Event Sourcing, CQRS, and Saga for the asynchronous event-based style. It is the ideal platform to plug in your business cross-cutting concerns to make it a customized platform for your particular industry.

If your organization can standardize the platform, then it is the most cost-effective approach. The engineering team can concentrate their efforts and won’t be distracted by varying heterogeneous platforms. In this case, a chassis-based approach is the best option.

If your organization is too big and wants to support multiple languages for microservices, then you can choose the second option with service mesh on the network level. However, the amount of effort to support an extra language to build services is immense. If your organization has less than 4000 engineers, it is recommended to use only one language. The number is calculated based on the number of engineers and official languages in Google.

The third option that combines chassis and service mesh is not recommended, especially if you have to build your own chassis. It combines most of the cons from the other two options without much benefit. If you are ever going that approach, it is better to pick up an open-source chassis to do so.

In the long run, it would be better to have a hybrid approach supported by an open-source platform that integrates with the service mesh natively. In the Light, we have several middleware handlers that talk to CA Layer 7 and RedHat 3 Scale gateways. We are also working on the light-portal exposes service mesh API to integrate with Istio mixers.