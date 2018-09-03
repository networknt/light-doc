---
title: "OpenAPI 3.0"
date: 2018-09-03T11:57:33-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

OpenAPI 3.0 was released in mid 2017 and we had support for it at the end of 2017. The specification is not backward compitable to the previous Swagger 2.0 and the tool chain around this specification format is still lacking. We have to build our own openapi-parser in order to support it in light-rest-4j framework. 

There are some middleware handlers designed for OpenAPI 3.0 specification. 

### OpenAPI 3.0 Specific Handlers

* [openapi-meta][] loads the openapi.yaml or openapi.json from the config folder during server start and caches the specification per endpoint.
* [openapi-security][] verifies the JWT token signature and expiration as well as scopes if they are defined in the specification.
* [openapi-validator][] validates the request headers, path parameters, query parameters and body against the specification.

[openapi-meta]: /style/light-rest-4j/openapi-meta/
[openapi-security]: /style/light-rest-4j/openapi-security/
[openapi-validator]: /style/light-rest-4j/openapi-validator/
