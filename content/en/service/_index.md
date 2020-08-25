---
title: "Service Overview"
date: 2017-11-04T22:12:19-04:00
description: "There are some services as part of the infrastructure to support microservices"
categories: [service]
keywords: []
menu:
  docs:
    parent: "service"
    weight: 01
weight: 01	#rem
aliases: []
toc: false
draft: false
---

For organizations that are planning to migrate an existing monolithic application to microservices, the upfront infrastructure investment is very significant. Some essential services must be implemented before deploying a large number of services.

Usually, you can start building several services and deploy them to your existing data center; however, once you have a dozen services running with multiple instances, you will need supporting services for security, tracing, monitoring, metrics, etc.

The following are some of the important services recommended by the Light platform. Some of them are built by ourselves as existing products on the market cannot meet our requirement with microservices in the cloud. For example, light-oauth2 and light-portal. Other services are our picks of industry leaders, and the list is changing as the landscape of cloud-native computing is changing these days dramatically. 

[light-oauth2][] is our security service built on top of light-rest-4j to protect services accessed by the unauthorized client. It has a unique feature to facilitate the service to service communication with two tokens.

- [Why this OAuth 2.0 Provider](/service/oauth/why-this-oauth/)
- [Introduction](/service/oauth/introduction/)
- [Getting started](/getting-started/light-oauth2/)
- [Architecture](/service/oauth/architecture/)
- [Documentation](/service/oauth/service/)
  * [Authorization Code][] - Authenticate to OAuth 2.0 and get authorization code
  * [SPNEGO/Kerberos][] - Authenticate to OAuth 2.0 with SPNEGO/Kerberos
  * [Token Endpoint][] - Token endpoint of OAuth 2.0 provider
  * [Signing Endpoint][] - Securely exchange information between microservices
  * [Service Registration][] - Service registration endpoints
  * [Client Registration][] - Client registration endpoints
  * [User Management][] - User management endpoints
  * [Key Distribution][] - Public key certificate distribution
  * [Refresh Token][] - Refresh token service
  * [Provider Registration][] - Oauth provider server service
  * [Custom grant type][] - Client authenticated user grant type
  * [PKCE][] PKCE implementation
- [OpenID Connect][] OpenID Connect implementation
- [Authenticator](/service/oauth/authenticator/)
- [Deployment](/service/oauth/deployment/)
  * [Deploy to Kubernetes](/service/oauth/deployment/kubernetes/)
  * [Secure HazelCast](/service/oauth/deployment/hazelcast/)
- [Tutorial](/tutorial/oauth/)
- [Reference](/service/oauth/reference/)
- [COVID-19](/service/covid-19/)


[oauth-kafka][] is our enterprise OAuth 2.0 provider build on top of light-kafka and kafka streams for event sourcing and CQRS. It is scalalbe and integrated with light-portal. The light-oauth2 will be still available for users who want to put everything together on a destop or a laptop. The API interface is the same and serviceIds are slightly different. This component is available for enterprise customers and subscription services will be availabe from light-portal site. 

[light-portal][] is an API runtime management and marketplace. In the service mesh context, it is the control plane. 


[light-router][] is a proxy service that can help an external or a  legacy client to access cloud-native microservices by encapsulating consumer side cross-cutting concerns into a microservice as a distributed gateway. 



[light-proxy][] is a proxy service that can bring legacy REST API to the Light ecosystem by encapsulating all provider side cross-cutting concerns into a microservice as a distributed gateway. 



[Messaging service][] is provided by [Apache Kafka][] for high throughput and high availability. 



[Logging][] is provided by [ELK][] stack for centralized logging



[Metrics][] is provided by [InfluxDB][] or [Prometheus][] with [Grafana][] as front-end.



[Tracing][] is provided by [OpenTracing][] and [Jaeger][] Tracer and [SkyWalking][] is working in process. 


[light-oauth2]: /service/oauth/
[oauth-kafka]: /service/oauth-kafka/
[light-proxy]: /service/proxy/
[light-portal]: /service/portal/
[Messaging service]: /service/messaging/
[Apache Kafka]: https://kafka.apache.org/
[Logging]: /service/logging/
[ELK]: https://www.elastic.co/webinars/introduction-elk-stack
[Metrics]: /service/metrics/
[InfluxDB]: https://www.influxdata.com/
[Prometheus]: https://prometheus.io/
[Grafana]: https://grafana.com/
[Authorization Code]: /service/oauth/service/code/
[Token Endpoint]: /service/oauth/service/token/
[Service Registration]: /service/oauth/service/service/
[Client Registration]: /service/oauth/service/client/
[User Management]: /service/oauth/service/user/
[Key Distribution]: /service/oauth/service/key/
[Refresh Token]: /service/oauth/service/fresh-token/
[Provider Registration]: /service/oauth/service/provider/
[SPNEGO/Kerberos]: /service/oauth/service/spnego/
[Signing Endpoint]: /service/oauth/service/signing/
[PKCE]: /service/oauth/service/pkce/
[Custom grant type]: /service/oauth/service/custom/
[OpenID Connect]: /service/oauth/serivce/openid/
[light-router]: /service/router/
[Tracing]: /service/tracing/
[OpenTracing]: /service/tracing/open-tracing/
[Jaeger]: /service/tracing/jaeger/
[SkyWalking]: /service/tracing/skywalking/
[How to secure Hazelcast]: /service/oauth/deployment/hazelcast/
