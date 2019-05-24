---
title: "Account Management Tutorial"
date: 2017-12-07T20:15:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This example can be found at [light-example-4j][] and it is a more realistic example than Todolist.

Account Money Transfer example is build on light-4j, light-rest-4j and light-eventuate-4j which 
uses event sourcing and CQRS as major patterns to handle event process in multiple microservices. 

The application consists of loosely coupled components that communicate using events and leverages
eventual consistency, event-driven approach rather than using traditional distributed transaction.

These components can be deployed either as separate services or packaged as a monolithic application 
for simplified development and testing.


### This example illustrates several important concepts:

* How to decompose an application into microservices - as described below the application consists 
of several services. For example, bank accounts are managed by one service, money transfers by 
another service.

* Using an event-driven architecture to achieve data consistency - rather than using traditional 
distributed transaction to maintain database consistency, this application uses an eventually 
consistent, event-driven approach.

* Using event sourcing to implement the event-driven architecture - the domain logic consists of 
Domain-Driven Design (DDD) aggregates using event sourcing.

* Using Command Query Responsibility Segregation (CQRS) - update requests (HTTP POSTs and PUTs) 
and view requests (HTTP GETs) are handled by separate services.

* How event sourcing enables deployment flexibility - the application can either be deployed as 
a monolith or as microservices.


### Structure of the example

Modules:

* common - common module for the application, include DDD models and events for event sourcing. 

* command - command side common components, include command, command services

* query - query side common components, include query, query services

* e2etest - end to end test module


### There are several services:

* Customers Service - REST API for creating customers

* Accounts Service - REST API for creating accounts

* Transactions Service - REST API for transferring money

* Customers View Service - subscribes to events and updates query side materialized view. And it 
provides an API for retrieving customers

* Accounts View Service - subscribes to events and updates query side materialized view. And it 
provides an API for retrieving accounts



### Event work flow:

##### Customer/Account creation and deletion

Account Money Transfer example uses event sourcing to handle the customer/account creation and deletion:

* On customer command side, system sends the create/delete customer commands and applies create/delete customer events

* On customer view side, system subscribes the create/delete customer events by registering event handles. On the example, system will process event and save or delete customer to local database.

* On account command side, system sends the create/delete account commands and applies create/delete account events

* On account view side, system subscribes the create/delete account events by registering event handles. On the example, system will process event and save or inactive account on local database.

* On customer view side, system subscribes the delete account events by registering event handles, and will remove the customer/account relationship


![account1](/images/account1.png)


##### Money Transfer:

Account Money Transfer example uses event sourcing to handle money transfer transaction control

* On transaction command side, system sends money transfer command and applies the money transfer event with certain amount

* On account command side, system subscribes the money transfer event. System will verify the account balance based on the debit event.

* If the balance is not enough, system publishes AccountDebitFailedDueToInsufficientFundsEvent. Otherwise, system sends account debit event/account credit event.

* On transaction command side, if subscribed events are account debit event/account credit event, system will process event and publish creditRecorded event and debitRecorded event,
  if subscribed event is AccountDebitFailedDueToInsufficientFundsEvent, system will publish FailedDebitRecordedEvent.

* On account view side, if subscribed events are creditRecorded event and debitRecorded event, system will update local account balance and update the transaction status to COMPLETED.
  if subscribed event is FailedDebitRecordedEvent, system will update transaction status to FAILED.


![account2](/images/account2.png)


### Build and Test

To build and test the application, please follow [this document][]

[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/eventuate/account-management
[this document]: /tutorial/eventuate/account-management/test/

