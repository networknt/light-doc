---
title: Get Started
linktitle: Get Started Overview
description: Quick start and guides for staring your first microserivce on your preferred operating system.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
categories: [getting started,fundamentals]
keywords: [usage,docs]
menu:
  docs:
    parent: "getting-started"
    weight: 1
weight: 40	#rem
sections_weight: 40
draft: false
aliases: [/overview/introduction/]
toc: false
---

If you don't have JDK 8, Maven and Git installed, please prepare your [environment][]. 

There are several frameworks available to support different style of API building. If this
is the very first time you try to create an API, please try [restful][]. If you are unsure
which style you should choose for your project, please read [API category][].

### Getting Started

The easiest way to start your RESTful API project is from Swagger 2.0 specification or OpenAPI 
3.0 specification and here is a video to show you how to generate a project from Swagger spec.

[light-4j-getting-started](https://youtu.be/xSJhF1LcE0Q)


And here is the [step to step rest tutorial][] with one of them is [Swagger petstore][] and
another one is [OpenAPI petstore][]. 

If you are interested in GraphQL framework, please refer to [light-graphql-4j][].

We also have a RPC/Hybrid framework that support JSON RPC and you can find some info at
[light-hybrid-4j][].

For event-driven service, you might be interested in [light-eventuate-4j][].


### Examples

Anther way to start a new project is to copy from an existing example and expends it. This 
is not recommended but if you don't have IDL(Inteface Definition Language) specification like 
OpenAPI specification or your service has very special requirement that cannot be generated, you 
can find many example projects at a separate repo [light-example-4j][] and start by copying one 
of them.

There are folders in light-example-4j that contains examples:

* rest folder container examples built on top of light-rest-4j framework
* graphql folder contains examples built on top of light-graphql-4j framework
* hybrid folder contains examples built on top of light-hybrid-4j framework
* eventuate folder contains examples built on top of light-eventuate framework


### Tutorials

Follow the [microservices tutorial][] to build multiple services that communicate each other. The 
source code for the tutorial can be found at [https://github.com/networknt/light-example-4j][] 

* api_a is calling api_b and api_c
* api_b is calling api_d
* api_c is an independent service
* api_d is an independent service


[API category]: /style/category/
[environment]: /getting-started/environment/
[restful]: /getting-started/light-rest-4j/
[step to step rest tutorial]: /tutorial/rest/
[Swagger petstore]: /tutorial/rest/swagger/petstore/
[OpenAPI petstore]: /tutorial/rest/openapi/petstore/
[light-graphql-4j]: /getting-started/light-graphql-4j/
[light-hybrid-4j]: /getting-started/light-hybrid-4j/
[light-eventuate-4j]: /getting-started/light-eventuate-4j/
[light-example-4j]: https://github.com/networknt/light-example-4j
[microservices tutorial]: /tutorial/
