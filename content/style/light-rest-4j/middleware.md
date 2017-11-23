---
title: "Middleware handlers for Restful API"
date: 2017-11-22T22:00:20-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

There are two set of middleware handlers or plugins available to support Swagger 2.0 specification
and OpenAPI 3.0 specification. For handlers for Swagger 2.0, they are prefixed with "swagger-" and
for handlers designed for OpenAPI 3.0, they are prefixed with "openapi-". 

### Middleware Handlers for OpenAPI 3.0

- [openapi-meta][] loads the OpenAPI 3.0 openapi.json and split it to operation level so that it can be looked up when the request comes in.
- [openapi-security][] is responsible to verify JWT token signature and expiry as well as verify scope defined in OpenAPI 3.0 spec against the JWT token scope.
- [openapi-validator][] validates the request against the specification for the endpoint on path parameters, query parameters, headers and body. 

### Middleware Handlers for Swagger 3.0

- [swagger-meta][] loads the OpenAPI 3.0 openapi.json and split it to operation level so that it can be looked up when the request comes in.
- [swagger-security][] is responsible to verify JWT token signature and expiry as well as verify scope defined in OpenAPI 3.0 spec against the JWT token scope.
- [swagger-validator][] validates the request against the specification for the endpoint on path parameters, query parameters, headers and body. 


[openapi-meta]: /style/light-rest-4j/openapi-meta/
[openapi-security]: /style/light-rest-4j/openapi-security/
[openapi-validator]: /style/light-rest-4j/openapi-validator/
[swagger-meta]: /style/light-rest-4j/swagger-meta/
[swagger-security]: /style/light-rest-4j/swagger-security/
[swagger-validator]: /style/lgiht-rest-4j/swagger-validator/


