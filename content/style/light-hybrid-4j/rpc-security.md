---
title: "Rpc Security"
date: 2018-05-07T21:13:37-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This handler is part of the [light-hybrid-4j][] which is built on top of light-4j but focused on hybrid style of API only. 

It supports OAuth2 with JWT token distributed verification and can be extended to other authentication and authorization approaches. 

### JwtVerifyHandler

This is the handler that is injected during server startup if security.yml enableVerifyJwt is true. It does further scope verification if enableVerifyScope is true against hybrid schema specification.

### Distributed JWT verification

Unlike simple web token that the resource server has to contact Authorization server to validate the bearer token. JWT can be verified by resource server as long as the token signing certificate is available at resource server. 

[light-hybrid-4j]: /style/light-hybrid-4j/

