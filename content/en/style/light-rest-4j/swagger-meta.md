---
title: "Swagger Meta"
date: 2017-11-22T22:45:49-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


This handler is part of the [light-rest-4j][] which is built on top of light-4j but 
focused on RESTful API only. Also, it only works with Swagger 2.0 specification. 

It is designed based on swagger specification so it is our best interest to utilize 
the swagger.json to its full potential. Currently there are two components are using 
the swagger specification during runtime.

1. [swagger-security][] - Verify scope in the JWT token against scope defined in swagger specification if scope verification is true. 
2. [swagger-validator][] - Validate request and response based on the definition in swagger specification for the uri and method.

As you have noticed, both components need to have swagger operation available 
based on the current request uri and method combination.

### Cache

A specification file swagger.json should be in the config folder of your API 
implementation and it will be loaded to memory with SwaggerHelper during server 
start up. It will be cached in memory until the server is restarted.

### Normalized Path

In order to match the incoming request path to the paths defined in the swagger 
specification, all paths are normalized before matching action. SwaggerHelper 
provides an API to match the request path to the paths in swagger specification.

### SwaggerHandler

This is an HttpHandler to parse the swagger spec based on the request uri and 
method and attach an SwaggerOperation object to the exchange. The swagger-security 
and swagger-validator modules are using it to do their jobs without parsing the 
swagger.json second time.

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[swagger-security]: /style/light-rest-4j/swagger-security/
[swagger-validator]: /style/light-rest-4j/swagger-validator/
