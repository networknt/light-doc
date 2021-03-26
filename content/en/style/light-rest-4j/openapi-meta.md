---
title: "OpenAPI Meta"
date: 2017-11-22T22:46:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This handler is part of [light-rest-4j][] which is built on top of light-4j but is focused on RESTful API only. It only works with the OpenAPI 3.0 specification.

It is designed based on the OpenAPI specification, so it is our best interest to utilize the openapi.yaml or openapi.json to its full potential. Currently, two components are using the OpenAPI specification during runtime.

1. [openapi-security][] - Verifies scopes in the JWT token against the scopes defined in OpenAPI specification if scope verification is true.
2. [openapi-validator][] - Validates request and response based on the definition in the OpenAPI specification for the path and method.

As you have noticed, both components need to have the OpenAPI operation available based on the current request path and method combination.

### Cache

A specification file openapi.yaml or openapi.json should be in the config folder of your API implementation, and it will be loaded to memory with OpenApiHelper during server startup. It will be cached in memory until the server is shut down.

### Normalized Path

To match the incoming request path to the paths defined in the OpenAPI specification, all paths are normalized before matching actions. OpenApiHelper provides an API to match the request path to the paths in OpenAPI specification.

### OpenApiHandler

This is an HttpHandler to parse the OpenAPI spec based on the request path and method and attach an OpenApiOperation object to the exchange. The openapi-security and openapi-validator modules use it to do their jobs without parsing the openapi.yaml or openapi.json a second time.

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[openapi-security]: /style/light-rest-4j/openapi-security/
[openapi-validator]: /style/light-rest-4j/openapi-validator/
