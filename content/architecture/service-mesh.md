---
title: "Service Mesh"
date: 2017-11-11T08:43:18-05:00
description: "A service mesh is a dedicated infrastructure layer for making service-to-service communication safe, fast, and reliable."
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
---

A service mesh is a dedicated infrastructure layer for making service-to-service communication 
safe, fast, and reliable. If you’re building a cloud native application, you need a service mesh!

Over the past year, the service mesh has emerged as a critical component of the cloud native stack. 
High-traffic companies like Paypal, Lyft, Ticketmaster, and Credit Karma have all added a service 
mesh to their production applications. But what is a service mesh, exactly? And why is it suddenly 
relevant?

In this article, I’ll define the service mesh and trace its lineage through shifts in application 
architecture over the past decade. I’ll distinguish the service mesh from the related, but distinct, 
concepts of API gateways, edge proxies, and the enterprise service bus. Finally, I’ll describe where 
the service mesh is heading, and what to expect as this concept evolves alongside cloud native adoption.

## What is a service mesh

A service mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s 
responsible for the reliable delivery of requests through the complex topology of services that comprise 
a modern, cloud native application. In practice, the service mesh is typically implemented as an array 
of lightweight network proxies that are deployed alongside application code, without the application 
needing to be aware. But there are variations to this idea, as it can be implemented in an API framework
so that it is embedded into the service application for maximum efficiency. 

The concept of the service mesh as a separate layer is tied to the rise of the cloud native application. 
In the cloud native model, a single application might consist of hundreds of services; each service might 
have thousands of instances; and each of those instances might be in a constantly-changing state as they 
are dynamically scheduled by an orchestrator like Kubernetes. Not only is service communication in this 
world incredibly complex, it’s a pervasive and fundamental part of runtime behavior. Managing it is vital 
to ensuring end-to-end performance and reliability.

## Is the service mesh a networking model?

The service mesh is a networking model that sits at a layer of abstraction above TCP/IP. It assumes that 
the underlying L3/L4 network is present and capable of delivering bytes from point to point. (It also 
assumes that this network, as with every other aspect of the environment, is unreliable; the service mesh 
must therefore also be capable of handling network failures.)

In some ways, the service mesh is analogous to TCP/IP. Just as the TCP stack abstracts the mechanics of 
reliably delivering bytes between network endpoints, the service mesh abstracts the mechanics of reliably 
delivering requests between services. Like TCP, the service mesh doesn’t care about the actual payload or 
how it’s encoded. The application has a high-level goal (“send something from A to B”), and the job of the 
service mesh, like that of TCP, is to accomplish this goal while handling any failures along the way.

Unlike TCP, the service mesh has a significant goal beyond “just make it work”: it provides a uniform, 
application-wide point for introducing visibility and control into the application runtime. The explicit 
goal of the service mesh is to move service communication out of the realm of the invisible, implied 
infrastructure, and into the role of a first-class member of the ecosystem—where it can be monitored, 
managed and controlled.

## What does a service mesh actually do?

Reliably delivering requests in a cloud native application can be incredibly complex. A service mesh like 
[light-eventuate-4j][] manages this complexity with a wide array of powerful techniques: latency-aware load 
balancing, eventually consistency, service discovery, retries, distributed transaction and deadlines. 
These features must all work in conjunction, and the interactions between these features and the complex 
environment in which they operate can be quite subtle.

It’s important to note that these features are intended to provide both pointwise resilience and 
application-wide resilience. Large-scale distributed systems, no matter how they’re architected, have one 
defining characteristic: they provide many opportunities for small, localized failures to escalate into 
system-wide catastrophic failures. The service mesh must be designed to safeguard against these escalations 
by shedding load and failing fast when the underlying systems approach their limits.


## Why is the service mesh necessary

The service mesh is ultimately not an introduction of new functionality, but rather a shift in where 
functionality is located. Web applications have always had to manage the complexity of service communication. 
The origins of the service mesh model can be traced in the evolution of these applications over the past 
decade and a half.

Consider the typical architecture of a medium-sized web application in the 2000’s: the three-tiered app. In 
this model, application logic, web serving logic, and storage logic are each a separate layer. The 
communication between layers, while complex, is limited in scope—there are only two hops, after all. There 
is no “mesh”, but there is communication logic between hops, handled within the code of each layer.

When this architectural approach was pushed to very high scale, it started to break. Companies like Google, 
Netflix, and Twitter, faced with massive traffic requirements, implemented what was effectively a 
predecessor of the cloud native approach: the application layer was split into many services (sometimes 
called “microservices”), and the tiers became a topology. In these systems, a generalized communication 
layer became suddenly relevant, but typically took the form of a “fat client” library—Twitter’s Finagle, 
Netflix’s Hystrix, and Google’s Stubby being cases in point.

In many ways, libraries like Finagle, Stubby, and Hystrix were the first service meshes. While they were 
specific to the details of their surrounding environment, and required the use of specific languages and 
frameworks, they were forms of dedicated infrastructure for managing service-to-service communication, and 
(in the case of the open source Finagle and Hystrix libraries) found use outside of their origin companies.

Fast forward to the modern cloud native application. The cloud native model combines the microservices 
approach of many small services with two additional factors: containers (e.g. Docker), which provide 
resource isolation and dependency management, and an orchestration layer (e.g. Kubernetes), which abstracts 
away the underlying hardware into a homogenous pool.

These three components allow applications to adapt with natural mechanisms for scaling under load and for 
handling the ever-present partial failures of the cloud environment. But with hundreds of services or 
thousands of instances, and an orchestration layer that’s rescheduling instances from moment to moment, the 
path that a single request follows through the service topology can be incredibly complex, and since 
containers make it easy for each service to be written in a different language, the library approach is no 
longer feasible.

This combination of complexity and criticality motivates the need for a dedicated layer for 
service-to-service communication decoupled from application code and able to capture the highly dynamic 
nature of the underlying environment. This layer is the service mesh.

## The future of the servie mesh

While service mesh adoption in the cloud native ecosystem is growing rapidly, there is an extensive and 
exciting roadmap ahead still to be explored. The requirements for serverless computing (e.g. Amazon’s 
Lambda) fit directly into the service mesh’s model of naming and linking, and form a natural extension 
of its role in the cloud native ecosystem. The roles of service identity and access policy are still very 
nascent in cloud native environments, and the service mesh is well poised to play a fundamental part of 
the story here. Finally, the service mesh, like TCP/IP before it, will continue to be pushed further into 
the underlying infrastructure. Just as light-eventuate-4j evolved from systems like Finagle, the current 
incarnation of the service mesh as a separate, user-space proxy that must be explicitly added to a cloud 
native stack will also continue to evolve.

## Conclusion

The service mesh is a critical component of the cloud native stack. A little more than half year from its 
launch, light-eventuate-4j is part of the core service layer for the light platform and almost all other
services are built on top of it. 

The light-eventuate-4j open source community of adopters and contributors are demonstrating the value of 
the service mesh model every day. We’re committed to building an amazing product and continuing to grow 
our incredible community. [Join us][]!



[Join us]: https://github.com/networknt/light-eventuate-4j
[light-eventuate-4j]: /style/light-eventuate-4j/


