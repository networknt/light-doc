---
title: "Light Gateway"
date: 2022-04-04T20:35:41-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

With more and more users implementing light-router to bring the legacy clients and light-proxy to bring the legacy services to the light platform, we added many features to both services. Most of the new features are actually from the traditional monolithic gateway products required by our customers. Since we have implemented most of the gateway features and it still has its usage in an enterprise environment, we thought it would be good to create a gateway based on microservices architecture. The new service is called light-gateway, and it is open-sourced in the networknt organization in GitHub. 

Like the http-sidecar, the light-gateway combines both light-proxy and light-router features in a standalone service that can be deployed as a Docker container, Windows or Linux service. 

Please be aware that all the features listed here are applicable to the [http-sidecar][]. 

The following are features we added to the light-gateway: 

* [Multiple OAuth 2.0][] provider support on the JWT token retrieval and JWT token verification via JWK. 
* [Rate Limit] helps the gateway regulate the traffic based on server, client, address, and user with optional request paths or services. 



[http-sidecar]: /service/http-sidecar/
[Multiple OAuth 2.0]: /service/gateway/multiple-oauth/
[Rate Limit]: /concern/limit/


