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

When building an application based on microservices architecture, there are two patterns
for service to service communication. 

* Synchronous - request/response
* Asynchronous - event driven

For large scale application, event driven approach gives you more flexibility and scalability
and request from client don't need all the services to be up and running the exact time. 

In addition to event driven application, Event Sourcing and CQRS are getting populate these
days for high scale applications. 

light-eventuate-4j is a framework that provides user to build Event Sourcing and CQRS services
without worrying about the underline infrastructure. It provides an event store and APIs to
publish and subscribe events to/from clients. 



