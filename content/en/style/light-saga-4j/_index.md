---
title: "Saga - Distributed Transactions"
date: 2017-12-07T12:44:55-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
---

In microservices architecture, it is recommended that each service has its own database. Some business 
transactions, however, span multiple service so you need a mechanism to ensure data consistency across 
services. How to maintain data consistency across services? In microservices architecture, we don't have
the luxury of XA transaction. 

The answer is Saga which is a sequence of local transactions. Each local transaction updates the database 
and publishes a message or event to trigger the next local transaction in the saga. If a local transaction 
fails because it violates a business rule then the saga executes a series of compensating transactions that 
undo the changes that were made by the preceding local transactions.

