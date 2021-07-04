---
title: "Event Sourcing"
date: 2018-01-06T15:34:45-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Event sourcing is an event-centric approach to persistence; A service that uses event 
sourcing persists each aggregate as a sequence of events. When it creates or updates an 
aggregate, the service saves one or more events in the database, which is also known as 
the event store. It reconstructs the current state of an aggregate by loading the events 
and replaying them. In functional programming terms, a service reconstructs the state of 
an aggregate by performing a functional fold/reduce over the events. Because the events 
are the state, you no longer have the problem of atomically updating state and publishing 
events.

For the service needs to use the event sourcing, the service should include light-eventuate-4j 
and implement it own event handler. The implemented event handler will call the API in 
light-eventuate-4j to process/subscriber the events.

