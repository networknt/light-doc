---
title: "Restful Database Access Tutorial"
date: 2017-11-23T14:23:52-05:00
description: ""
categories: []
keywords: [tutorial]
menu:
  docs:
    parent: "tutorial"
    weight: 20
weight: 20
aliases: []
toc: false
draft: false
---


Most microservices will have to access database in order to fulfill consumer requests. 
In this tutorial, we will walk through the following steps with Oracle/Postgres/Mysql
for light-rest-4j framework. If you use light-graphql-4j or light-hybrid-4j the steps
will be somewhat different but the concept is the same. Please note, the specification
we use in this tutorial is based on Swagger 2.0

* How to setup database connection pool
* How to connect to the database instance
* How to do query database tables
* How to update database tables

This tutorial shows how to use the SQL database in your microservices based on light*4j
frameworks. 

* [Environment preparation][]

* [Create specification][]

* [Build and start][]

* [Test service][]

* [Enable security][]

* [Docker][]

* [Metrics][]

* [Logging][]

* [Summary][]

[Environment preparation]: /tutorial/rest/swagger/database/preparation/
[Create specification]: /tutorial/rest/swagger/database/specification/
