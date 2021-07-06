---
title: "Todo-list Tutorial"
date: 2017-12-07T20:14:36-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


This is the most straightforward Hello World type of application with Event Sourcing and CQRS on top of light-eventuate-4j. It has command side service that adds a new todo item, updates an existing todo item and removes a todo item. It also has query side service to retrieve all the todo items for display in a single page application. 

This example can be found at [light-example-4j][]. 

### There are five parts in this projects:

* Common module defines domain object and event object across modules on both command side and query side

* Command side API implemented on top of light-eventuate-4j to build and publish events.

* Command side microservice (Restful based or hybrid based) to trigger command API

* Query side API implemented on top of light-eventuate-4j to define the event handlers and subscribe events and process by specified event handlers.

* Query side microservice (Restful based or hybrid based) to trigger query API.


### Process Flow

![drawing5](/images/Drawing5.png)

Step 1: Application or browser sends restful HTTP post request to command service to create a todo

Step 2: Command service processes the request and generates a "create todo" event; And then publishes the event to the event store

Step 3: CDC service will capture the data change based on MySQL replication log and publish the event data to Kafka distributed stream system

Step 4: Query side service's event handler  knows the new event data published to Kafka, an event handler will subscribe to the event

Step 5: Query side service will process the event and save the result to a local table. (In this example: mysql/todo_db/TODO)

Step 6: Application or browser sends restful HTTP GET request to query service to get ALL todo list

Step 7: Query side service processes the request and gets the todo list from local table

Step 8: Query side service returns the HTTP response to the consumer.

### Steps

The following steps will assume you know the basics about light-eventuate-4j as well as light-rest-4j and light-hybrid-4j as we are going to build services in both RESTful style and Hybrid RPC style. 

Before starting any service, we need to make sure that light-eventuate-4j is up and running. Please follow this [getting started][] to set up.

We are going to implement the application in both light-rest-4j and light-hybrid-4, and the two implementations will be sitting in the same folders in light-example-4j. 

As the todo-list folder already in the public repository of light-example-4j/eventuate, we are going to rename the existing project so that you can compare these folders if you want.

Here are all steps in this tutorial and it is the best to follow them in sequence.

* [Prepare Envionment][]

* [Create Project][]

* [Rest Command Side][]

* [Rest Query Side][]

* [Rest Test][]

* [Hybrid Command Side][]

* [Hybrid Query Side][]

* [Hybrid Test][]

* [Rest Docker][]

* [Hybrid Docker]

[getting started]: /tutorial/eventuate/getting-started/
[Prepare Envionment]: /tutorial/eventuate/todo-list/prepare/
[Create Project]: /tutorial/eventuate/todo-list/project/
[Rest Command Side]: /tutorial/eventuate/todo-list/rest-command/
[Rest Query Side]: /tutorial/eventuate/todo-list/rest-query/
[Rest Test]: /tutorial/eventuate/todo-list/rest-test/
[Hybrid Command Side]: /tutorial/eventuate/todo-list/hybrid-command/
[Hybrid Query Side]: /tutorial/eventuate/todo-list/hybrid-query/
[Hybrid Test]: /tutorial/eventuate/todo-list/hybrid-test/
[Rest Docker]: /tutorial/eventuate/todo-list/rest-docker/
[Hybrid Docker]: /tutorial/eventuate-4j/todol-list/hybrid-docker/
[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/eventuate/todo-list


