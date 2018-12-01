---
title: "Messaging Infrastructure"
date: 2017-11-18T16:29:00-05:00
description: ""
categories: [service]
keywords: [oauth2]
menu:
  docs:
    parent: "service"
    weight: 20
weight: 20	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

When you make architecture decision on how to communicate between service to service, chances are you want to use the event-driven approach instead of request/response approach. It gives you the flexibility, reliability, and scalability.
 
To leverage the full potential of the light platform, you should use different frameworks for different services. Some of the services serve the application API calls from SPA and mobile applications, then these services will be built on top of light-rest-4j, light-graphql-4j or light-hybrid-4j to serve client synchronously. There are cooperations and orchestrations between these services, and event-based asynchronous approach is more suitable. Also, there are a lot of processes in the backend are event-based in nature and they need to be processed in streams which can attach microservices along the pipe. 

Once the decision is made, the next big one is to choose which framework to help to manage the interaction between services. In the light platform, light-eventuate-4j is an Event Sourcing and CQRS framework built on top of light-4j and utilize Kafka for messaging broker by default. 

For some special use cases, you might consider using light-blockchain-4j as your platform for private or public chain applications. It is based on the Kafka broker at the moment. 

The light platform is very flexible in term of injecting other components, and we are exploring other messaging brokers as well. We are working with Solace closely to integrate its messaging system with light-eventuate-4j. 

- [Kafka](/service/messaging/kafka/)
  * [Installation](/service/messaging/kafka/installation/)
  * [Security](/service/messaging/kafka/security/)
  * [Purge Topic](/service/messaging/kafka/purge/)
- [Schema Registry](/service/messaging/schema-registry/)
  * [Installation](/service/messaging/schema-registry/installation/)
  * [Security](/service/messaging/schema-registry/security/)
