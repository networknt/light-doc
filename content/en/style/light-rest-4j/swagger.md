---
title: "Swagger 2.0"
date: 2018-09-03T11:57:38-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Swagger 2.0 is the first release of the Swagger specification that is used to describe RESTful APIs. There are some gaps in the specification but the tool chain around this specification is the most mature. As OpenAPI 3.0 is new, most existing project are using Swagger 2.0 specification format. For brand new project, it is recommended using OpenAPI 3.0.

There are some middleware handlers designed for Swagger 2.0 specification. 

### Swagger 2.0 Specific Handlers

* [swagger-meta][] loads the swagger.json from the config folder during server start and caches the specification per endpoint.
* [swagger-security][] verifies the JWT token signature and expiration as well as scopes if they are defined in the specification.
* [swagger-validator][] validates the request headers, path parameters, query parameters and body against the specification.


[swagger-meta]: /style/light-rest-4j/swagger-meta/
[swagger-security]: /style/light-rest-4j/swagger-security/
[swagger-validator]: /style/light-rest-4j/swagger-validator/
