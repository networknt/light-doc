---
title: "Eventuate - Event Sourcing and CQRS"
date: 2017-12-07T12:44:15-05:00
description: ""
categories: [style]
keywords: []
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
---

When building an application based on microservices architecture, there are two major patterns
for service to service communication. 

### Serivce to Service communication

##### Synchronous, Request/Response Communication

In light-4j, this means to use Client module to call other services in the request/response
fashion regardless light-rest-4j, light-graphql-4j or light-hybrid-4j is used. 
 
##### Asynchronous, Message-Based Communication

The eventual consistency framework light-eventuate-4j is designed to facilitate asynchronous
communication between services built on top of light-rest-4j, light-grahpql-4j and light-hybrid-4j.

Service communication is through events and every service maintain its own aggregates to serve 
consumer independently.
 
### Which Style to Choose

For pros and cons of each communication patterns, please see [communication pattern][]


### Why light-eventuate-4j 

For microservices implementation, developing business transactions that update entities that are 
owned by multiple services is a challenge, as is implementing queries that retrieve data from multiple 
services; For complicated application that has too many services interact with each other. It is
essential to choose asynchronous message-based style for service communication and transaction
management with eventual consistency. 

For large scale application, event driven approach gives you more flexibility and scalability
and requests from client don't need all the services to be up and running the exact time. 

In addition to event driven application, Event Sourcing and CQRS are getting populate these
days for high scale applications. 

Once you've decided that asynchronous message-based communication is best for your use case, you
need to make some architecture decision to choose the best of combination of frameworks to build
your microservices.

For large scale application, event driven approach gives you more flexibility and scalability
and requests from client don't need all the services to be up and running the exact time. 

In addition to event driven application, Event Sourcing and CQRS are getting populate these
days for high scale applications. 

light-eventuate-4j is a framework that provides user to build Event Sourcing and CQRS services
without worrying about the underline infrastructure. It provides an event store and APIs to
publish and subscribe events to/from clients. You can build your services with light-rest-4j, 
light-graphql-4j or light-hybrid-4j and they are all seemlessly integrated with light-eventuate-4j 
framework which provide event store and event producing and consuming client for services built 
on top of other light-4j frameworks.

Another alternative solution would be [light-tram-4j][] which is another event driven framework but
without the complicated event sourcing to deal with. 

### light-eventuate-4j components:

- light-4j, light-rest-4j, light-graphql-4j and light-hybrid-4j provide platforms to build microservices in REST, Graphql and Hybrid/RPC style.

- Mysql, PostgreSql or Oracle for event store to persist events

- Zookeeper is used to manage Kafka cluster

- Apache Kafka is used as message brokers for publishing/subscribing events

- A console/admin UI to monitor the message broker and communication between services

- A hybrid command server to host all hybrid command services in case light-hybrid-4j is used

- A hybrid query server to host all hybrid query services in case light-hybrid-4j is used
 
- A RESTful CDC service that is responsible to publish events to Kafka from event store if light-rest-4j is used.
  
### Architecture

Here is a list of architecture decisions for light-eventuate-4j:

* [Communication Pattern][] 

* [Event Driven][] 

* [Event Sourcing][]

* [CQRS][]

* [Why not use Kafka as source of truth][]

### Design

* [Concepts][]
* [Duplicate Events][]



[communication pattern]: /style/light-eventuate-4j/comm-pattern/
[light-tram-4j]: /style/light-tram-4j/
[Event Driven]: /style/light-eventuate-4j/event-driven/
[Event Sourcing]: /style/light-eventuate-4j/event-sourcing/
[CQRS]: /style/light-eventuate-4j/cqrs/
[Why not use Kafka as source of truth]: /style/light-eventuate-4j/kafka-eventstore/
[Concepts]: /style/light-eventuate-4j/concepts/
[Duplicate Events]: /style/light-eventuate-4j/dup-events/


