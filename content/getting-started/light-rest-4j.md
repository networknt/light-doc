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
reviewed: true
---

The light-rest-4j is a framework that is designed to speed up RESTful API development and deployment. On top of light-4j, it has several middleware handlers specifically designed around Swagger 2.0 and OpenAPI 3.0 specifications. With the specification ready, you can scaffold project with [light-codegen][], and the specification will be loaded during the runtime to enable JWT scope verification and schema validation for the request based on the spec. 

### Start a project with specification and light-codegen

The light-4j framework is a [design-driven][] framework which means you have to start with the specification before starting with code. It makes sense as the contract is the most important thing for a service and it should be finalized before coding is started. The easy way to start a microservice with the light-rest-4j framework is to use [light-codegen][] to scaffold a running application based on your Swagger 2.0 specification or OpenAPI 3.0 specification and a config.json that controls how code generator works.

At the moment, we support both the Swagger 2.0 specification and the OpenAPI 3.0 specification. Both specs are not the same, and they are not backward compatible, so we have to create two sets of middleware handlers to address them separately.

If you don't know which specification to use, please follow the OpenAPI 3.0 tutorial as it is the future.

* [Swagger 2.0 Petstore Tutorial][]
* [OpenAPI 3.0 Petstore Tutorial][]
* [OpenAPI 3.0 Petstore Kotlin Tutorial][]

For other tutorials on Restful services, please refer to [tutorial][]. 

### Start a project from an existing example

Another way to start a brand new project is to copy from one of the existing examples. You can find several Restful API examples at [light-example-4j][] repository and the majority of them has a [tutorial][] to walk you through step by step. If you are interested in implementing your service in Kotlin, you can find examples in [light-example-kotlin][] repository. 


[light-codegen]: /tool/light-codegen/
[Swagger 2.0 Petstore Tutorial]: /tutorial/rest/swagger/petstore/
[OpenAPI 3.0 Petstore Tutorial]: /tutorial/rest/openapi/petstore/
[OpenAPI 3.0 Petstore Kotlin Tutorial]: /tutorial/rest/openapi/petstore-kotlin/
[tutorial]: /tutorial/rest/
[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/rest
[light-example-kotlin]: https://github.com/networknt/light-example-kotlin/tree/master/openapi
[design-driven]: /design/design-first/


