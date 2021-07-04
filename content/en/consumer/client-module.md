---
title: "Client Module"
date: 2018-03-01T21:33:09-05:00
description: ""
categories: []
keywords: []
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

The [client module][] in light-4j is a very important component in Light for
interactions between client to service or service to service in synchronous request/response
over HTTPS and HTTP2. Microservices are expected to have high throughput and low latency
so that multiple services can be chained together to fulfill one external request. This is
why the client module is designed for direct connection between the consumer and provider with
the HTTP 2.0 over TLS connection cached on the consumer side for most high throughput applications.
For low throughput consumers, the connection can be closed after the request and reopen when
the next request comes in. 

There are a lot of features in the client module and these should be utilized in certain
scenarios. The following section shows how to use the client module for common use cases
and the detail usage is documented here as well. 


- [TLS Connection](/consumer/tls-connection/)
- [HTTP 2.0](/consumer/http2/)
- [OAuth 2.0 JWT Token](/consumer/oauth2-jwt/)
  * [Subject Token](/consumer/subject-token/)
  * [Access Token](/consumer/access-token/)
  * [Customized Grant Type](/consumer/customized-grant/)
  * [Get Token in Startup](/consumer/token-startup/)
  * [Long Lived Token](/consumer/long-lived-token/)


[client module]: /concern/client/