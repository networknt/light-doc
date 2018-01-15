---
title: "Messaging Infrastructure"
date: 2017-11-18T16:29:00-05:00
description: ""
categories: [service]
keywords: [oauth2]
menu:
  docs:
    parent: "service"
    weight: 20
weight: 20	#rem
aliases: []
toc: false
draft: false
---

When you make architecture decision on how to communicate between service to service, chances are you
want to use event-driven approach instead of request/response approach. This gives you the flexibility
, reliability and scalability.
 
Once the decision is made, the next big one is to choose which framework to help to manage the interaction
between services. In light platform, light-eventuate-4j is an Event Sourcing and CQRS framework built on
top of light-4j and utilize Kafka for messaging broker by default. 


 