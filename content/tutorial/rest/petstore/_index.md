---
title: "Petstore"
date: 2017-11-06T17:45:44-05:00
description: "How to build petstore API from specification"
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
and in this tutorial, we will walk you through the step to get petstore generated
and then deployed to production. Within these steps, we will show you how to use
the features of light-4j and light-rest-4j frameworks.

Please note that our frameworks are aiming microservices and there are several
tutorial to show case multiple services interact with each other. These can be
found at [rest tutorial][] section. The public available petstore specification
is designed as monolithic API. 


[Petstore][] is a generated RESTful API project based on OpenAPI specification 
found [here][]. After the project is generated, we will try to change the configuration 
to enable other features in following steps.


This tutorial shows how to use the service registry and discovery feature of light-*-4j
frameworks. 

* [Environment preparation][]

* [Code generation][]

* [Build and start][]

* [Test service][]

* [Enable security][]

* [Docker][]

* [Metrics][]

* [Logging][]

* [Summary][]





[rest tutorial]: /tutorial/rest/
[Petstore]: https://github.com/networknt/light-example-4j/tree/master/rest/petstore
[here]: https://github.com/networknt/model-config/tree/master/rest/petstore/2.0.0
[Environment preparation]: /tutorial/rest/petstore/environment/
[Code generation]: /tutorial/rest/petstore/generate/
[Build and start]: /tutorial/rest/petstore/build/
[Test service]: /tutorial/rest/petstore/test/
[Enable security]: /tutorial/rest/petstore/security/
[Docker]: /tutorial/rest/petstore/docker/
[Metrics]: /tutorial/rest/petstore/metrics/
[Logging]: /tutorial/rest/petstore/logging/
[Summary]: /tutorial/rest/petstore/summary/

