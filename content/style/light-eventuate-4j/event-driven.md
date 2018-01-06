---
title: "Event Driven Architecture"
date: 2018-01-06T15:34:35-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

An Eventuate application consists of four types of modules, each with different roles and responsibilities:

* Command-side module
* Query-side view updater module
* Query-side view query module
* Outbound gateway module

Note that this is the logical architecture of the application. The modules can either be deployed together 
as a monolithic application or separately as standalone microservices.

### Command-side modules

The command-side of an Eventuate application consists of one or more command-side modules. A command-side 
module creates and updates aggregates in response to external update requests (for example, HTTP POST, PUT, 
and DELETE requests) and events published by command-side aggregates. A command-side module consists of 
following components:

* Aggregates - Implement most of the business logic and are persisted using event sourcing

* Services - Process external requests by creating, updating and deleting aggregates

* Event handlers - Process events by creating, updating and deleting aggregates

### Query-side modules

The query-side of an Eventuate application maintains one or more materialized (CQRS) views of command-side 
aggregates. Each view has two modules:

* View updater module - Subscribes to events and updates the view

* View query module - Handles query (for example, HTTP GET) requests by querying the view

### Outbound gateway modules

An outbound gateway module processes events by invoking external services.

Diagram:


![drawing3](/images/Drawing3.png)


### Light-4j based eventuate architecture

In an event-driven architecture, a service publishes events when something notable happens, such as when it 
updates a business object. Other services subscribe to those events. In response to an event a service typically 
updates its own state. It might also publish more events, which then get consumed by other services

![drawing1](/images/Drawing1.png)



