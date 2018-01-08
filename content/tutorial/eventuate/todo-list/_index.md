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
---


This is the most simple Hello World type of application with event sourcing
and CQRS on top of light-eventuate-4j. It has command side service that add
new todo item, update existing todo item and remove todo item(s). It also
has query side service to retrieve all the todo items for display in the web
single page application. 

This example can be found at [light-example-4j][]. 

There are five parts in this projects:

* common module define domain object and event object across module both command side and query side

* Command side API implemented on top of light-eventuate-4j to build and publish events.

* Command side microservice (Restful based or hybrid based) to trigger command API

* Query side API implemented on top of light-eventuate-4j to define the event handles and subscribe events and process by defined event handles.

* Query side microservice (Restful based or hybrid based) to trigger query API.


The following steps will assume you know the basic about light-eventuate-4j
as well as light-rest-4j and light-hybrid-4j as we are going to build services
in RESTful style and Hybrid RPC style. 

Before starting any service, we need to make sure that light-eventuate-4j is
up and running. Please follow this [getting started][] to set up.


We are going to implement the application in both light-rest-4j and light-hybrid-4
and the two implementations will be sitting in the same folders in light-example-4j. 

As the todo-list folder already in the public repo of light-example-4j/eventuate, we 
are going to rename the existing project and you can compare these folders if you want.

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


