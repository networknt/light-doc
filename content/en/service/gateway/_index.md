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

The following are some ideas to manage the light-gateway.

* [Path Prefix Naming Convention][] can help us manage hundreds of APIs on the same gateway.
* [Header Sanitization][] to prevent XSS injection for X-Correlation-Id and X-Traceability-Id.

The following are features we added to the light-gateway: 

* [Multiple OAuth 2.0][] provider support on the JWT token retrieval and JWT token verification via JWK. 
* [Rate Limit][] helps the gateway regulate the traffic based on server, client, address, and user with optional request paths or services. 
* [Path Prefix Service][] middleware handler supports routeing traffic based on the request paths to different backend APIs.
* [SkipVerifyScopeWithoutSpec][] flag to avoid scope verification from JWT for backend API endpoints in JwtVerifyHandler.
* [UnifiedSecurityHandler][] to combine Basic, OAuth and ApiKey together for the same request path. 
* [Generic Transformer][] to transform the request and response before sending to the downstream API or return to the caller.

The following are the products with different configurations. 

* [light-gateway][] to replace the traditional monolithic gateway shared by multiple consumers and providers. 
* [light-proxy-client][] to bring legacy consumers to the light ecosystem with all the cross-cutting concerns
* [light-balancer][] to enable the high availability of light-proxy-client (LPC) for multiple consumers on the same host.
* [light-proxy-server][] to bring legacy providers to the light ecosystem with all the cross-cutting concerns. 
* [external-access-point][] to wrap one or more internal or external APIs and expose a single API to the third-party consumers.

[http-sidecar]: /service/http-sidecar/
[Multiple OAuth 2.0]: /service/gateway/multiple-oauth/
[Rate Limit]: /concern/limit/
[Path Prefix Service]: /service/gateway/path-prefix/
[light-proxy-client]: /service/gateway/light-proxy-client/
[light-gateway]: /service/gateway/light-gateway/
[light-balancer]: /service/gateway/light-balancer/
[light-proxy-server]: /service/gateway/light-proxy-server/
[SkipVerifyScopeWithoutSpec]: /service/gateway/skip-scope/
[UnifiedSecurityHandler]: /service/gateway/unified-security/
[Generic Transformer]: /service/gateway/generic-transformer/
[external-access-point]: /service/gateway/external-access-point/
[Path Prefix Naming Convention]: /service/gateway/naming-convention/
[Header Sanitization]: /service/gateway/header-sanitization/
