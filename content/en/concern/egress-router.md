---
title: "Egress Router"
date: 2021-11-07T21:53:53-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The egress-router module is used by the [http-sidecar][] and [light-router][] to address cross-cutting concerns for the outgoing traffic. It depends on some of the middleware handlers from light-4j or light-rest-4j in the request and response chain. Besides, we have implemented several middleware handlers for light-router and http-sidecar specific requirements. These handlers are located in the light-4j repository and test cases are located in the light-router repository. Some of them are mandatory, and some of them are optional. 

* [TokenHandler](/service/router/token-handler/)
* [PathServiceHandler](/service/router/path-service/)
* [PathPrefixServiceHandler](/service/router/path-prefix-serivce/)
* [ServiceDictHandler](/service/router/service-dict/)

[http-sidecar]: /service/http-sidecar/
[light-router]: /service/router/
