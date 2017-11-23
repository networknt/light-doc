---
title: "Swagger Security"
date: 2017-11-22T22:45:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This handler is part of the [light-rest-4j][] which is built on top of light-4j but 
focused on RESTful API only. Also, it only works with Swagger 2.0 specification.

It supports OAuth2 with JWT token distributed verification and can be extended to 
other authentication and authorization approaches. 

### JwtVerifyHandler

This is the handler that is injected during server start up if security.yml 
enableVerifyJwt is true. It does further scope verification if enableVerifyScope 
is true against swagger specification.

### Distributed JWT verification

Unlike simple web token, the resource server has to contact Authorization server 
to validate the bearer token. JWT can be verified by resource server as long as 
the token signing certificate is available at resource server. 


[light-rest-4j]: https://github.com/networknt/light-rest-4j


