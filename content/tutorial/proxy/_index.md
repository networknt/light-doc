---
title: "Tutorial of light-proxy"
date: 2017-12-16T09:36:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-proxy is a very important service component that can glue all other alien APIs into the
light platform ecosystem without re-writting. Once putting light-proxy in front of legacy APIs
these API can enjoy all the cross-cutting concerns provided by the light-platform, such as security,
metrics, auditing, traceability, validation, service registry/discovery etc. 

There are two separate tutorials here to leverage either Swagger 2.0 specification or OpenAPI 3.0
specification.  

### Swagger 2.0 Backend

* [Swagger backend service][] to simulate legacy RESTful service built with Swagger 2.0 spec.

* [Swagger proxy][] to set up light-proxy in front of Swagger 2.0 service

### OpenAPI 3.0 Backend 

* [OpenAPI backend service][] to simulate legacy RESTful service built with OpenAPI 3.0 spec. 

* [OpenAPI proxy][] to set up light-proxy in front of OpenAPI 3.0 service

[Swagger backend service]: /tutorial/proxy/swagger-backend/
[Swagger proxy]: /tutorial/proxy/swagger-proxy/
[OpenAPI backend service]: /tutorial/proxy/openapi-backend/
[OpenAPI proxy]: /tutorial/proxy/openapi-proxy/

