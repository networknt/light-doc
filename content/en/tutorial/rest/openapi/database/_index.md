---
title: "Restful Database Access"
date: 2017-11-23T14:23:52-05:00
description: "How to access database from Restful API"
categories: []
keywords: [tutorial]
aliases: []
toc: false
draft: false
reviewed: true
---

There are numerous options to connect to databases in the Light platform, and this tutorial is only one of the most straightforward options. It gives you the flexibility to support multiple databases with just configuration changes. However, you are working with generic JDBC configuration and losing fine-grained control on each specific database JDBC driver. If you are using one particular database and want more settings on the JDBC driver, please use the [data-source][] module in light-4j.

Also, for large-scale microservices deployment, you are seldom dealing with the database directly from your service. Chances are your service is working with in-memory cache or in-memory data grid or some No-SQL database directly. For service to service orchestration, you might take a look at the [light-eventuate-4j][] to adopt Event Sourcing and CQRS as your architecture pattern. 

In this tutorial, we will walk through the following steps with Oracle/Postgres/Mysql for light-rest-4j framework. If you use light-graphql-4j or light-hybrid-4j frameworks, the steps will be somewhat different, but the concept is the same. Please note, the specification we use in this tutorial is based on OpenAPI 3.0, which is migrated from the Swagger 2.0. 

* How to use [HikariCP](https://github.com/brettwooldridge/HikariCP) for JDBC connection pool.
* How to initialize database connection pool and plug it into light-4j startup hooks.
* How to connect to the database instance.
* How to query database tables.
* How to update database tables.
* How to performance test API application with wrk.

This tutorial shows how to use the SQL database in your microservices-based on light-4j frameworks. 

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

* [Switch to Oracle][]

* [Switch to Postgre][]

* [Unit tests][]

* [Performance][]

[Environment preparation]: /tutorial/rest/openapi/database/preparation/
[Create specification]: /tutorial/rest/openapi/database/specification/
[Generate and Build]: /tutorial/rest/openapi/database/generation/
[Test mock service]: /tutorial/rest/openapi/database/test/
[Database scripts]: /tutorial/rest/openapi/database/dbscripts/
[Start databases]: /tutorial/rest/openapi/database/startdb/
[Connection pool]: /tutorial/rest/openapi/database/connection-pool/
[Single query]: /tutorial/rest/openapi/database/single-query/
[Multiple queries]: /tutorial/rest/openapi/database/multiple-queries/
[Multiple updates]: /tutorial/rest/openapi/database/multiple-updates/
[Unit tests]: /tutorial/rest/openapi/database/unit-test/
[Performance]: /tutorial/rest/openapi/database/performance/
[data-source]: /concern/datasource/
[light-eventuate-4j]: /style/light-eventuate-4j/
[Switch to Oracle]: /tutorial/rest/openapi/database/oracle/
[Switch to Postgre]: /tutorial/rest/openapi/database/postgres/
