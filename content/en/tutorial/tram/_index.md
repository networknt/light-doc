---
title: "Tram Tutorial"
date: 2017-12-19T18:51:47-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Light-tram-4j is the foundation of event driven frameworks and handles transactional
messaging without XA. On top of light-tram-4j, there is light-saga-4j to handle distributed 
transactions across multiple microservices. There is also a framework light-eventuate-4j to
support Event Sourcing and CQRS architecture. All tutorials here are showing basic message,
event or command delivery from one service to another service. 

### [Todo-list][]

This is a Hello World type of application that shows the features of light-tram-4j. Unlike
the [todo-list implemented in light-eventuate-4j][] with Event Sourcing and CQRS, this demo
application will only use message/event from a service that is responsible for inserting, 
updating and deleting todo items to another service that is responsible for query todo items.
 


### [Development][]

Different from light-evenutate-4j, Light-tram-4j handles message delivery in a lighter way and
doesn't have event sourcing and CQRS. Light-tram-4j doesn't handle event replay and doesn't worry 
about aggregate status.

Light-tram-4j  can be simply used in those services which handle its local database process and 
send message to message broker together.


[todo-list]: /tutorial/tram/todo-list/
[todo-list implemented in light-eventuate-4j]: /tutorial/eventuate/todo-list/
[Development]: /tutorial/tram/develop/