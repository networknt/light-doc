---
title: "Hybrid - Modularized Monolithic"
date: 2017-12-20T21:21:09-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
---

For most organizations, switching from monolithic architecture to microservices is really hard as 
the initial infrastructure investment is very high and almost all the design patterns in Java EE 
are anti-patterns in microservices. It might be easier to start with modularized monolithic and then
split each module to separate instances as microservices once volume picks up. light-hybrid-4j is
designed for it. It takes advantages of both monolithic and microservices with simple RPC style of
interaction between services.

Light-hybrid-4j contains the following components.

### rpc-router

rpc-router is a middleware handler that implement light-4j [handler interface][] and it will be inject
into the request/response chain. It is responsible for parsing the incoming request body and then
route to the request to the right handler to handle it. The handler will be implemented in one of
the service jar file included into the server instance. 


### rpc-security

rpc-security is responsible for JWT token verification from Authorization header.  

[handler interface]: /concern/handler/


- [JSON Protocol](/style/light-hybrid-4j/json-rpc/)
  * [Schema Definition](/style/light-hybrid-4j/json-schema/)
  * [Code Generator](/tutorial/generator/)
  * [Start Server](/style/light-hybrid-4j/start-server/)
  * [Test Service](/style/light-hybrid-4j/test-service/)
  * [Tutorial](/tutorial/hybrid/)
- [Binary Protocol](/style/light-hybrid-4j/binary-rpc/)
