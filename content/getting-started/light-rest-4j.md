---
title: "Light Rest 4j"
date: 2017-11-05T11:46:06-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 10
weight: 10
sections_weight: 10
aliases: []
toc: false
draft: false
---

Light-rest-4j is a framework that is designed to speed up RESTful API development and 
deployment. On top of light-4j, it has several middleware handlers specifically designed 
around Swagger 2.0 and OpenAPI 3.0 specifications. With the specification ready, you can 
scaffold project with [light-codegen][] and the specification will be loaded during the 
runtime to to enable JWT scope verification and schema validation for the request. 

### Start a project with specification and light-codegen

light-4j framework is a design driven framework which means you have to start with the
specification before start with code. This make sense as the contract is the most important
thing for services and it should be finalized before coding is started. Once the specification
is ready, [light-codegen][] can be utilized to generate the project. 

At the moment, we support both Swagger 2.0 spec and OpenAPI 3.0 spec. Both specifications are 
not the same and they are not backward compatible so we have to create two sets of middleware 
handlers to address them separately.

An easy way to start with RESTful API is to follow the petstore tutorials. If you don't
know which specification to use, please follow the OpenAPI 3.0 tutorial as it is the
future.

* [Swagger 2.0 Petstore Tutorial][]
* [OpenAPI 3.0 Petstore Tutorial][]

For ohter tutorials on Restful services, please refer to [tutorial][]. 

### Start a project from an existing example

Another way to start a brand new project is to copy from one of the existing examples. 

You can find several Restful API examples at [light-example-4j][] repository and majority of 
them has a [tutorial][] to walk you through step by step.

The applications that have a matching tutorial are usually multiple services interact with 
each other. There is a separate example petstore which is described above.


[light-codegen]: /tool/light-codegen/
[Swagger 2.0 Petstore Tutorial]: /tutorial/rest/swagger/petstore/
[OpenAPI 3.0 Petstore Tutorial]: /tutorial/rest/openapi/petstore/
[tutorial]: /tutorial/rest/
[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/rest

