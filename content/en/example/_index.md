---
title: "Example Overview"
date: 2017-11-06T17:45:44-05:00
description: "Examples"
categories: []
keywords: [example]
menu:
  docs:
    parent: "example"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Starting with examples is a good way to learn how to use each frameworks. Some of the examples
have a corresponding tutorials so that you can follow them to rebuilt the examples step by step.

Most examples are in [light-examples-4j][] repository in networknt organization on github.com;
however, there are some other examples scattered in other organizations or repositories. 


### Database

##### Restful Database Access 

This example can be found at [rest/swagger/database][] and there is a [database tutorial][] for it.

There are three databases (Mysql, Postgres and Oracle) are used in this example and you can switch
database with just configuration change in service.yml

The example shows:

* How to use [HikariCP](https://github.com/brettwooldridge/HikariCP) for JDBC connection pool.
* How to initialize database connection pool and plug it into light-4j startup hooks.
* How to dockerize light-4j application.
* How to compose databases with light-4j application so that you can start all servers together.
* How to performance test API application with wrk.
* Postgres performs better than mysql on my i5 desktop.

##### Techempower Performance Benchmark

We have submitted benchmark implementations to Techempower to compare light-4j with other frameworks
on the market. The source code can be found in [techempower repo][] and it is very good example on
how to build high performance services including database access. The benchmark result can be found
at [techempower benchmark][] 


### Microservices

There are several example applications that have multiple microserivces and they can be found in 
https://github.com/networknt/light-example-4j/tree/master/rest

These examples will show you: 

* How to build microservices
* How to do API to API call with light-4j client component
* How to protect API with JWT token with scopes
* How to performance test APIs with wrk
* How to centralize logs with ELK
* How to collect metrics for each services
* How to deployed with Kubernetes

##### Microservices Chain Pattern - Swagger

This example can be found at [/rest/swagger/ms_chain][] and there is a [chain tutorial][] for it.

In this example, there are four services chained together. A calls B, B calls C and C calls D in a
chain pattern. 

##### Microservices Branch Pattern

Coming soon!

##### Microservices Aggregate Pattern - OpenAPI

This example can be found at [/rest/openapi/ms_aggregate][] and you can use the above **chain tutorial Quick Start** link to understand how to run it as they are executed similarly. For a step-by-step tutorial you can visit [aggregate tutorial][].



### Performance 

light-4j and other related light-*-4j frameworks are some of the fastest frameworks to
build microservices on the market in different styles. There are several examples that
show developers how to implement high performance microservices.

##### Microserivces Benchmark

There is a [benchmark repository][] within networknt organization to compare all popular microservices
frameworks on the market. The test is done on a desktop computer and focus on the raw throughput and 
latency of each framework. There might be more test cases added to the suite in the future though. 


##### TechEmpower Benchmark

TechEmpower maintains a benchmark to compare all web frameworks and the source code can be found at 
[techempower repo][]. The result can be found at [techempower benchmark][]. We submitted our tests
but there are still a lot of room to optimize the result especially the multiple database query and 
multiple database updates. 


### Routing

This is an example to show you how to use undertow routing handler. It opens
the door to customize it and put your own handlers in front of common handlers
provided.

The source code repo can be found at [routing repository][]


### Web Server


### Websocket




[light-examples-4j]: https://github.com/networknt/light-example-4j
[rest/swagger/database]: https://github.com/networknt/light-example-4j/tree/master/rest/swagger/database
[database tutorial]: /tutorial/rest/swagger/database/
[techempower repo]: https://github.com/TechEmpower/FrameworkBenchmarks/tree/master/frameworks/Java/light-java
[techempower benchmark]: https://www.techempower.com/benchmarks/previews/round15/
[/rest/swagger/ms_chain]: https://github.com/networknt/light-example-4j/tree/master/rest/swagger/ms_chain
[/rest/openapi/ms_aggregate]: https://github.com/networknt/light-example-4j/tree/master/rest/openapi/ms-aggregate
[chain tutorial]: /tutorial/rest/swagger/ms-chain/
[aggregate tutorial]: /tutorial/rest/openapi/ms-aggregate/
[benchmark repository]: https://github.com/networknt/microservices-framework-benchmark
[routing repository]: https://github.com/networknt/light-example-4j/tree/master/routing

