---
title: "Transaction Management"
date: 2017-11-06T11:14:26-05:00
description: "Transaction management in microservices architecture"
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
reviewed: true
---

In traditional monolithic applications, we normally use distributed transactions([XA](https://en.wikipedia.org/wiki/X/Open_XA))
and two-phase commit([2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)) 
to maintain data consistency and integrity.

After moving to microservices architecture, we don't have the option to use 
XA correctly technically; however, a lot of developer will use it implicitly 
without knowing the risk. In fact, they don't even realize that they are using 
distributed transactions between microservices.

## What is a distributed transaction

When we are talking about distributed transactions in the monolithic world, we are talking about explicit distributed transactions. In microservices architecture people will not use explicit distributed transactions but they can use implicit distributed transactions if they are not very careful.

Graham Lea defines distributed transaction in his article - [Distributed Transactions: The Icebergs of Microservices](http://www.grahamlea.com/2016/08/distributed-transactions-microservices-icebergs/)
as: 

Distributed transaction - any situation where a single event results in the 
mutation of two or more separate sources of data which cannot be committed 
atomically.

## Risk of distributed transactions in microservices

A lot of developers use implicit distributed transactions between multiple microservices. It works most of the time, until one day, there is a small network glitch and the data of your application is messed up as the application designer only thinks about happy paths. 

This happens when one command from the UI triggers two or more microservices to update their states. If everything is perfect, all updates will be successful and everybody is happy. But what if the first service update succeeds and the second fails due to a network issue? Is this acceptable in your business? The majority of business people will answer no.

Given the fact that we are doing microservices, how do we resolve the data consistency issue between services.? There are solutions based on how these services interact with each other.


## Solution - request/response 

When service to service communication is synchronous, the options are very limited. Graham Lea documented some solutions in his article and I have written some related articles before.

- [Idempotency of Microservices][]
- [How to handler partial failure][] 

## Solution - event based

A microservices architecture promotes availability over consistency, hence leveraging 
a distributed transaction coordinator with a two-phase commit protocol is usually not
an option. It gets even more complicated with the idea that these loosely-coupled data 
repositories might not necessarily all be relational stores and could be a polyglot 
with a mix of relational and non-relational data stores. The idea of having an immediately 
consistent state is difficult. What you should look forward to, however, is an architecture 
that promotes eventual consistency like CQRS and Event Sourcing. An event driven architecture 
can support a BASE (basic availability, soft-state, and eventual consistency) transaction 
model. This is a great technique to solve the problem of handling transactions across 
services and customers can be convinced to negotiate the contract towards a BASE model since 
usually transactional consistency across functional boundaries is mostly about frequent 
change of states, and consistency rules can be relaxed to let the final state be eventually 
visible to the consumers.

To enable developers to use an event driven approach for service to service communication, we have provided [light-eventuate-4j](https://github.com/networknt/light-eventuate-4j) which supports Event Sourcing and CQRS with Kafka as event broker. You can build your service with light-rest-4j, light-graphql-4j or light-hybrid-4j. They will communicate with each other asynchronously with events and eventually the states of each service will be consistent.

## XA in one service

We’ve talked above about transactions across multiple microservices. How about XA transactions in one service? For example, in one service, it updates two different databases in one transaction or updates one database and pushes a message into the MQ queue in the same transaction.

In general, it is OK to use XA in one service and for light-*-4j frameworks, you can use a third party library to manage XA transactions even if we are not on the Java EE platform.

[Transactions Essentials](https://www.atomikos.com/Main/TransactionsEssentials)
is an open source transaction manager and it is free. 

Although it is doable with a Java SE JTA/XA transaction manager, it is not scaling at all.
For high performance microservices, it would be better to adapt eventual consistency instead. 

When discussing transactions between multiple microservices, light-eventuate-4j is recommended for resolving data consistency between services. In one service, it can still be used to eliminate the need for XA transactions.

## Complex business transaction

For complex business transactions, achieving only eventual consistency is not enough. Saga pattern should be used to ensure that a partial of services can be rolled back with compensation steps if one of the services fails.

Our saga implementation is still a work in progress. Here is an [article](https://blog.bernd-ruecker.com/saga-how-to-implement-complex-business-transactions-without-two-phase-commit-e00aa41a1b1b) explaining it well.

[Idempotency of Microservices]: /design/idempotency/
[How to handler partial failure]: /design/partial-failure/
