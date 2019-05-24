---
title: "Swagger 2.0 Petstore"
date: 2017-11-06T17:45:44-05:00
description: "How to build petstore API from Swagger 2.0 specification"
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
the features of light-4j and light-rest-4j Swagger 2.0 support. If you are interested 
in light-rest-4j OpenAPI 3.0 support, please take a look at [OpenAPI 3.0 Petstore Tutorial][].

Please note that our frameworks are aiming microservices and there are several
tutorials to show case multiple services interact with each other. These can be
found at [rest tutorial][] section. The public available petstore specification
is designed as monolithic API. 


[Petstore][] is a generated RESTful API project based on Swagger 2.0 specification 
found [here][]. After the project is generated, we will try to change the configuration 
to enable other features in following steps.


This tutorial shows how to use the service registry and discovery feature of light*4j
frameworks. 

1. [Environment preparation][]

2. [Code generation][]

3. [Build and start][]

4. [Test service][]

5. [Enable security][]

6. [Docker][]

7. [Compose][]

8. [Logging][]

9. [Metrics][]

10. [Summary][]


[rest tutorial]: /tutorial/rest/
[Petstore]: https://github.com/networknt/light-example-4j/tree/master/rest/swagger/petstore
[here]: https://github.com/networknt/model-config/tree/master/rest/swagger/petstore/2.0.0
[Environment preparation]: /tutorial/rest/swagger/petstore/environment/
[Code generation]: /tutorial/rest/swagger/petstore/generate/
[Build and start]: /tutorial/rest/swagger/petstore/build/
[Test service]: /tutorial/rest/swagger/petstore/test/
[Enable security]: /tutorial/rest/swagger/petstore/security/
[Docker]: /tutorial/rest/swagger/petstore/docker/
[Metrics]: /tutorial/rest/swagger/petstore/metrics/
[Logging]: /tutorial/rest/swagger/petstore/logging/
[Summary]: /tutorial/rest/swagger/petstore/summary/
[OpenAPI 3.0 Petstore Tutorial]: /tutorial/rest/openapi/petstore/
[Compose]: /tutorial/rest/swagger/petstore/compose/
