---
title: "Tram Tutorial"
date: 2017-12-19T18:51:47-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 60
weight: 60
aliases: []
toc: false
draft: false
---

Light-tram-4j is the foundation of event driven frameworks and it handles transactional
messaging without XA. On top of light-tram-4j, there is light-saga-4j to handle distributed 
transactions across multiple microservices. Also there is a framework light-eventuate-4j to
support Event Sourcing and CQRS architecture. All tutorials here are showing basic message,
event or command delivery from one service to another service. 

### [todo-list][]

This is a Hello World type of application that shows the features of light-tram-4j. Unlike
the [todo-list implemented in light-eventuate-4j][] with Event Sourcing and CQRS, this demo
application will only use message/event from a service that is responsible for inserting, 
updating and deleting todo items to another service that is responsible for query todo items.
 


[todo-list]: /tutorial/tram/todo-list/
[todo-list implemented in light-eventuate-4j]: /tutorial/eventuate/todo-list/

### [development][]

Different as light-evenutate-4j, Light-tram-4j is light way to handle message deliver. Light-tram-4j doesn't handle event replay and doesn't worry about aggregate status.
Light-tram-4j  can be simply used in those services which handle its local database process and send message to message broker together.




[develop]: /tutorial/tram/develop/