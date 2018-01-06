---
title: "Todo list tutorial"
date: 2017-12-18T21:23:56-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

It's challenging for micro-service API to update data (e.g. a Domain-Driven design aggregate) and 
publish a message, such as a domain event. The traditional approach of using 2PC/JTA isn't a good 
fit for micro-service applications.

The light-tram-4j framework implements an alternative mechanism based on the Application Events 
pattern.

When an application creates or updates data, as part of that ACID transaction, it inserts an event 
into a MESSAGE table. A separate process publishes those events to a message broker, such as Apache 
Kafka.

### About the Todo list application

The Todo List application, which lets users maintain a todo list, is the end-to-end POC application 
for the light-tram-4j framework.

It shows how to use light-tram-4j to:

  * reliably publish domain events as part of a database transaction that updates an aggregate.

  * consume domain events to update a CQRS view
  
When a user creates or updates a todo on the command side service, the application publishes a 
domain event. An event handler on the query side service, subscribes to those events and updates 
an ElasticSearch-based CQRS view.


### Todo list architecture


The Todo List application is built using

Java JDK 8

light-4j

light-tram-4j

Kafka/Zookeeper

MySql

ElasticSearch


The application persists the Todo entity in MySQL. It also maintains a materialized view of the 
data in ElasticSearch.


### How it works


The Todo application uses the light-tram-4j framework to publish and consume domain events.

1. TodoCommandService persist todo entity to local data store and publishes an event when it 
creates, updates, or deletes a Todo item. It uses the DomainEventPublisher, which is implemented 
by the light-tram-4j framework.

2. light-tram-4j cdc server will get the published events from MESSAGE table and publish to Kafka 
message broker.

3. TodoQueryService application use TodoEventConsumer defines the event handlers, which update 
Elasticsearch to maintain a materialize view.


### Tutorial

The following steps are included in the tutorial and there are simple steps in [README.md][]


* [Prepare environment][]

* 



[README.md]: https://github.com/networknt/light-example-4j/tree/master/tram/light-tram-todolist/multi-module
[Prepare environment]: /tutorial/tram/todo-list/prepare/


