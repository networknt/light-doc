---
title: "Openapi Meta"
date: 2017-11-22T22:46:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This handler is part of the [light-rest-4j][] which is built on top of light-4j but 
focused on RESTful API only. Also, it only works with OpenAPI 3.0 specification. 

It is designed based on OpenAPI specification so it is our best interest to utilize 
the openapi.json to its full potential. Currently there are two components are using 
the OpenAPI specification during runtime.

1. [openapi-security][] - Verify scope in the JWT token against scope defined in OpenAPI specification if scope verification is true. 
2. [openapi-validator][] - Validate request and response based on the definition in OpenAPI specification for the uri and method.

As you have noticed, both components need to have OpenAPI operation available 
based on the current request uri and method combination.

### Cache

A specification file openapi.json should be in the config folder of your API 
implementation and it will be loaded to memory with OpenApiHelper during server 
start up. It will be cached in memory until the server is restarted.

### Normalized Path

In order to match the incoming request path to the paths defined in the OpenAPI 
specification, all paths are normalized before matching action. OpenApiHelper 
provides an API to match the request path to the paths in OpenAPI specification.

### OpenApiHandler

This is an HttpHandler to parse the OpenAPI spec based on the request uri and 
method and attach an OpenApiOperation object to the exchange. The openapi-security 
and openapi-validator modules are using it to do their jobs without parsing the 
openapi.json second time.

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[openapi-security]: /style/light-rest-4j/openapi-security/
[openapi-validator]: /style/light-rest-4j/openapi-validator/
