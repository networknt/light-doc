---
title: "Light Hybrid 4j"
date: 2017-11-05T11:46:42-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 30
weight: 30
sections_weight: 30
aliases: []
toc: false
draft: false
---

For public APIs, it makes sense to use RESTful, however, if it is an internal API,
some sort of RPC based API style will be more efficient. As these days, Javascript on
the browser are really powerful and it can deal with JSON or other binary protocol
very easily. These browser object (JSON/Binary) contains type information so that there
is no need to do Object to URI and URI to Object transformation based on OpenAPI 
specification or RAML.

If you look at an OpenAPI specification, you will be shock as it is too complicated with
so many moving parts. Both client and server will be dealing with headers, query parameters,
path parameters and request body. Also, because of the transformation between JSON object
to URI, some type information is lost and you have to defined it in the specification so
that the server can derive the type from it. This makes REST framework less efficient than
other RPC based framework with just send the JSON or binary object to the server with all
the type information available. 

With OpenAPI specification, it is very easy to combine several services together as merging
schema is much easier then OpenAPI spec. This is way we call the light-hybrid-4j framework
as hybrid. You can put several services together into the same JVM instance to save memory
and later on you can split up high volume service to separate JVM instance for scaling. This
give customer most flexible deployment options and they can move from monolithic->hybrid->
microservices.

Light-hybrid-4j is a generic RPC framework and it supports JSON RPC and binary RPC like gRPC
or other binary protocol later. 

The easiest way to start a hybrid service is two use [light-codegen][] to generate a server
and then generate a service based on a schema. Then you can put the service in the pom.xml
of the server or put the service jar file into the classpath when starting the server. 

- [Hybrid Framework](/style/light-hybrid-4j/) 
- [JSON Protocol](/style/light-hybrid-4j/json-rpc/)
  * [Schema Definition](/style/light-hybrid-4j/json-schema/)
  * [Code Generator](/tutorial/generator/)
  * [Start Server](/style/light-hybrid-4j/start-server/)
  * [Test Service](/style/light-hybrid-4j/test-service/)
  * [Tutorial](/tutorial/hybrid/)
- [Binary Protocol](/style/light-hybrid-4j/binary-rpc/)

[light-codegen]: /tool/light-codegen/
