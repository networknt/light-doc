---
title: "Eventuate Tutorial"
date: 2017-12-07T20:05:42-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 70
weight: 70
aliases: []
toc: false
draft: false
---

light-eventuate-4j framework is complex than any request/response based framework like
light-rest-4j, light-graphql-4j or light-hybrid-4j as there are more moving parts. In this
section, we are going to introduce some tutorials to help users to work with this framework.

Before start a service project based on light-eventuate-4j, you need to get the environment
set up first and the [getting started light-eventuate-4j][] has all the information to help
you to get infrastructure services started. 

Once you have your environment set up, the first tutorial is [todo-list][] which is a hello
world type of application to demonstrate how to build two microservices on command side and
query side based on Event Sourcing and CQRS. 

Todo-list is a simple application that demonstrates the features of light-eventuate-4j with 
Event Sourcing and CQRS. Although it is very simply, it is utilizing all the parts of the 
eventuate framework. It helps the developer to focus on the flow of the eventuate framework 
instead of complex business logic. If you want to see a full blown business application, then 
you can can refer to [account management tutorial][].


To help users to grasp other part of the framework we have the following tutorials to focus
each difficult areas. 

[Kafka Tutorial][] helps to work with Kafka in Docker container.

[Test Tutorial][] helps users to write unit tests, integration tests and end-to-end tests.

[Developer's Tutorial][] guidelines for developers who working on the framework or services


[account management tutorial]: /tutorial/eventuate/account-management/
[getting started light-eventuate-4j]: /tutorial/eventuate/getting-started/
[todo-list]: /tutorial/eventuate/todo-list/
[Kafka Tutorial]: /tutorial/eventuate/kafka/
[Test Tutorial]: /tutorial/eventuate/test/
[Developer's Tutorial]: /tutorial/eventuate/developer/
