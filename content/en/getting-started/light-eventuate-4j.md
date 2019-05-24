---
title: "Light Eventuate 4j"
date: 2017-11-05T11:46:59-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 40
weight: 40
sections_weight: 40
aliases: []
toc: false
draft: false
---

One of the major issues that we have seen occur in a system with microservices is the way transactions 
work when they span over different services. Within our old monolithic architecture, we have been using 
distributed transactions to resolve this, but they come with their own issues. It is very hard to avoid
deadlocks and scale the application when business grows. 

Distributed transaction management is a key architectural consideration that needs to be addressed whenever 
you are proposing a microservices deployment model to a customer. For developers from the monolithic 
relational database world, it is not an easy idea to grasp. In a monolithic application with a relational 
data store, transactions are ACID (atomic, consistent, isolated, and durable) in nature and follow a simple 
pessimistic control for data consistency. Distributed transactions are always guided by a two-phase commit 
protocol, where first all the changes are temporarily applied and then committed to, or rolled back, 
depending on whether or not all the transactions were successful or there was any error. You could easily 
write queries that fetched data from different relational sources and the distributed transaction coordinator 
model supported transaction control across disparate relational data sources.

In today's world, however, the focus is on modular, self-contained APIs. To make these services self-contained 
and aligned with the microservices deployment paradigm, the data is also self-contained within the module, 
loosely coupled from other APIs. Encapsulating the data allows the services to grow independently, but a major 
problem is dealing with keeping data consistent across the services. If every service has its own database,
then distributed transaction is not an option. 

A microservices architecture promotes availability over consistency, hence leveraging a distributed transaction
coordinator with a two-phase commit protocol is usually not an option. It gets even more complicated with the 
idea that these loosely-coupled data repositories might not necessarily all be relational stores and could be a 
polyglot with a mix of relational and NoSQL data stores. The idea of having an immediately consistent state 
is difficult. What you should look forward to, however, is an architecture that promotes eventual consistency 
like [CQRS][] and [Event Sourcing][]. An [event driven][] architecture can support a BASE (basic availability, 
soft-state, and eventual consistency) transaction model. This is a great technique to solve the problem of 
handling transactions across services and customers can be convinced to negotiate the contract towards a BASE 
model since usually transactional consistency across functional boundaries is mostly about frequent change of 
states, and consistency rules can be relaxed to let the final state be eventually visible to the consumers.


### Distributed data management problems in a microservice architecture:

In microservice system, normally each service has its own database. Some business transactions, 
however, span multiple services so you need a mechanism to ensure data consistency across 
services. For example, let’s imagine that you are building an e-commerce store where customers 
have a credit limit. The application must ensure that a new order will not exceed the customer’s 
credit limit. Since Orders and Customers are in different databases the application cannot simply 
use a local ACID transaction.

### Event-driven architecture

A good way to solve these distributed data management challenges is by using an event-driven 
architecture. In an event-driven application services publish and consume events. A service 
publishes an event whenever it changes the state of an entity. Another service can subscribe 
to that event and update its own entities and possibly publish other events. You can implement 
a business transaction that updates multiple entities as a series of steps, each of which 
updates one entity and publishes an event that triggers the next step. Also, a service can 
maintain the consistency of a replica by subscribing to the events that are published when 
the master copy is updated.

### Event Sourcing

One essential requirement in an EDA is the ability to atomically update state and publish events. 
In a traditional application the database storing the state and the message broker could participate 
in a distributed transactions. But the lack of 2PC in a cloud native world makes this a challenging 
problem to solve. 

A good solution to this problem is to use event sourcing. Event sourcing persists the state of 
a business entity as a sequence of state-changing events. Whenever the state of a business entity 
changes, a new event is appended to the list of events. Since saving an event is a single 
operation, it is inherently atomic. The application reconstructs an entity’s current state by 
replaying the events.

Applications save events in an event store, which is a database of events. The store has an API 
for adding and retrieving an entity’s events. The event store also behaves like a message broker. 
It provides an API that enables services to subscribe to events. When a service saves an event 
in the event store, it is delivered to all interested subscribers.

### light-eventuate-4j

When building services with light platform, you can use request/response approach
like [Rest][], [GraphQL][] or [Hybrid][] to serve consumers. For service to service
communication, you have two options. One is synchronous request/response over HTTP.
This requires all involved services to be responsive and highly available. 
There is another way to handler interaction between services - asynchronous event driven.
light-eventuate-4j is not only event driven framework, it supports Event Sourcing and
CQRS to ensure data consistency between services. 

If you are a developer to build services, then you don't need to clone the source code
of light-eventuate-4j. You can start all the services through a docker-compose. 

The details of the steps are documented in [light-eventuate-4j tutorial][]. 

### Some useful links

[Distributed Transactions: The Icebergs of Microserivces](http://www.grahamlea.com/2016/08/distributed-transactions-microservices-icebergs/)

[Transactions in Microservices](https://dzone.com/articles/transactions-in-microservices)

[Developing Transactional Microservices Using Aggregates, Event Sourcing and CQRS - Part 1](https://www.infoq.com/articles/microservices-aggregates-events-cqrs-part-1-richardson)

[Developing Transactional Microservices Using Aggregates, Event Sourcing and CQRS - Part 2](https://www.infoq.com/articles/microservices-aggregates-events-cqrs-part-2-richardson)


[light-docker]: https://github.com/networknt/light-docker
[Rest]: /style/light-rest-4j/
[GraphQL]: /style/light-graphql-4j/
[Hybrid]: /style/light-hybrid-4j/
[light-eventuate-4j tutorial]: /tutorial/eventuate/getting-started/
[CQRS]: /style/light-eventuate-4j/cqrs/
[Event Sourcing]: /style/light-eventuate-4j/event-sourcing/
[event driven]: /style/light-eventuate-4j/event-driven/

