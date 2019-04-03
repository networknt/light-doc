---
title: "OpenAPI 3.0 Petstore"
date: 2017-11-21T09:59:35-05:00
description: "How to build petstore API from OpenAPI 3.0 specification"
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

The petstore RESTful API is used by a lot of frameworks as reference application
and in this tutorial, we will walk you through the steps to get petstore generated
and then deployed to production. Within these steps, we will show you how to use
the features of light-4j and light-rest-4j OpenAPI 3.0 support. If you are
interested in light-rest-4j Swagger 2.0 support, please take a look at 
[Swagger 2.0 Petstore Tutorial][].

Please note that our frameworks are aiming microservices and there are several
tutorials to show case multiple services interact with each other. These can be
found at [rest tutorial][] section. The public available petstore specification
is designed as monolithic API. 


[Petstore][] is a generated RESTful API project based on OpenAPI 3.0 specification 
found [here][]. After the project is generated, we will try to change the configuration 
to enable other features in following steps.


This tutorial shows how to use the service registry and discovery feature of light*4j
frameworks. 

* [Environment preparation][]

* [Code generation][]

* [Build and start][]

* [Test service][]

* [Enable security][]

* [Docker][]

* [Compose][]

* [Logging][]

* [Metrics][]

* [Summary][]


[rest tutorial]: /tutorial/rest/
[Petstore]: https://github.com/networknt/light-example-4j/tree/master/rest/openapi/petstore
[here]: https://github.com/networknt/model-config/tree/master/rest/openapi/petstore/1.0.0
[Environment preparation]: /tutorial/rest/openapi/petstore/environment/
[Code generation]: /tutorial/rest/openapi/petstore/generate/
[Build and start]: /tutorial/rest/openapi/petstore/build/
[Test service]: /tutorial/rest/openapi/petstore/test/
[Enable security]: /tutorial/rest/openapi/petstore/security/
[Docker]: /tutorial/rest/openapi/petstore/docker/
[Metrics]: /tutorial/rest/openapi/petstore/metrics/
[Logging]: /tutorial/rest/openapi/petstore/logging/
[Summary]: /tutorial/rest/openapi/petstore/summary/
[Swagger 2.0 Petstore Tutorial]: /tutorial/rest/swagger/petstore/
[Compose]: /tutorial/rest/openapi/petstore/compose/
