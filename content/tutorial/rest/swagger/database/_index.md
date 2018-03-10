---
title: "Restful Database Access"
date: 2017-11-23T14:23:52-05:00
description: "How to access database from Restful API"
categories: []
keywords: [tutorial]
menu:
  docs:
    parent: "tutorial"
    weight: 30
weight: 30
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

* [Generate and Build][]

* [Test mock service][]

* [Database scripts][]

* [Start databases][]

* [Connection pool][]

* [Single query][]

* [Multiple queries][]

* [Multiple updates][]

* [Unit tests][]

* [Performance][]

[Environment preparation]: /tutorial/rest/swagger/database/preparation/
[Create specification]: /tutorial/rest/swagger/database/specification/
[Generate and Build]: /tutorial/rest/swagger/database/generation/
[Test mock service]: /tutorial/rest/swagger/database/test/
[Database scripts]: /tutorial/rest/swagger/database/dbscripts/
[Start databases]: /tutorial/rest/swagger/database/startdb/
[Connection pool]: /tutorial/rest/swagger/database/connection-pool/
[Single query]: /tutorial/rest/swagger/database/single-query/
[Multiple queries]: /tutorial/rest/swagger/database/multiple-queries/
[Multiple updates]: /tutorial/rest/swagger/database/multiple-updates/
[Unit tests]: /tutorial/rest/swagger/database/unit-test/
[Performance]: /tutorial/rest/swagger/database/performance/

