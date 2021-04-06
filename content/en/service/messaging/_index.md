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

When you make architecture decisions on how to communicate between service to service, chances are you want to use the event-driven approach instead of the request/response approach. It gives you flexibility, reliability, and scalability.

To leverage the full potential of Light, you should use different frameworks for different services. Some of the services serve the application API calls from SPA and mobile applications, then these services will be built on top of light-rest-4j, light-graphql-4j or light-hybrid-4j to serve clients synchronously. There is cooperation and orchestration between these services. An event-based asynchronous approach is more suitable. There are also a lot of processes in the backend which are event-based in nature and need to be processed in streams which can attach microservices along the pipe.

Once the decision is made, the next big one is to choose which framework to help to manage the interactions between services. In Light, light-eventuate-4j is an Event Sourcing and CQRS framework , which is built on top of light-4j, utilizes Kafka as a messaging broker by default.

For some special use cases, you might consider using light-blockchain-4j as your platform for private or public chain applications. It is based on the Kafka broker at the moment. 

Light is very flexible in terms of injecting other components, and we are exploring other messaging brokers as well. We are working with Solace closely to integrate its messaging system with light-eventuate-4j.

- [Kafka](/service/messaging/kafka/)
  * [Installation](/service/messaging/kafka/installation/)
  * [Security](/service/messaging/kafka/security/)
  * [Purge Topic](/service/messaging/kafka/purge/)
  * [Docker Access Local Kafka](/service/messaging/kafka/docker-local-kafka/)
  * [Upgrade Kafka Cluster](/service/messaging/kafka/upgrade/)
- [Schema Registry](/service/messaging/schema-registry/)
  * [Installation](/service/messaging/schema-registry/installation/)
  * [Security](/service/messaging/schema-registry/security/)
