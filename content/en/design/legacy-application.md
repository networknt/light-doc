---
title: "Legacy Application Integration with light-router and light-proxy"
date: 2021-11-09T12:18:24-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For any enterprise to implement an API platform in the cloud, they need to think about bringing legacy clients and legacy services into the ecosystem. Most organizations will have a lot of existing applications that need to consume APIs deployed to the Kubernetes cluster. Some of the applications can also provide REST APIs to allow other applications to leverage.

To help backend APIs to address cross-cutting concerns, we have developed the http-sidecar for services deployed to the Kubernetes cluster. It is suitable for any services deployed to the Kubernetes cluster; however, most legacy applications cannot be deployed to Kubernetes clusters. How can we bring these applications to the light-4j ecosystem? 


The http-sidecar is built on top of light-router and light-proxy to address cross-cutting concerns for outgoing and incoming HTTP requests in the same pod as the backend API. It is the combination of the features from light-router and light-proxy with sidecar configuration in mind. To integrate with legacy clients and services, we can use the light-router and light-proxy as standalone components deployed along with the legacy client applications or legacy service applications. 


The light-router and light-proxy can be used to address the following use cases. 


### Integrate both legacy consumer and legacy provider.

![heterogeneous-services](/images/heterogeneous-servics.png)

In the above diagram, both the consumer and provider are legacy applications. The light-router will bring the consumer to the ecosystem to manage the JWT token, discovery services etc. The light-proxy will bring the provider to the ecosystem to address request validation, service registration, security verification, etc. 


### Integrate legacy provider


![legacy-provider](/images/legacy-provider.png)


In this diagram, only the backend service is legacy. For example, it can be a database that exposes data through REST API like Neo4j or a third-party application with a special authentication mechanism like Tableau. In most cases, it might be a service we have been using for years and don't want to upgrade it. 

By putting a light-proxy instance in front of it, the provider will be like other services in the ecosystem. The location of the proxy is very flexible, it can be deployed to the same server as the legacy services, or it can be deployed to the Kubernetes cluster. We need to secure the channel if the proxy is not deployed locally with the legacy services. 

### Integrate legacy consumer


![legacy-consumer](/images/legacy-consumer.png)


In this diagram, we have a legacy consumer who wants to consume APIs in the light-4j ecosystem. The application might be from a third-party vendor or an application running for a long time. Instead of changing the application, we can put a light-router instance along with the application. The light-router will act on behalf of the consumer to get the token, cache the token, renew the token, discover services and address all other cross-cutting concerns for invoking APIs. 


### Router as External Access Point


![external-access](/images/external-access.png)


In this diagram, the light-router is acted as an external access point for third-party applications to consumer our services in the Kubernetes cluster. 

It will be responsible for authenticating/authorizing the request, rate limiting, token transform, service discovery, aggregation, etc.


### Router serves SPA


![serve-spa](/images/serve-spa.png)


In this diagram, the light-router serves the single-page application and addresses all the security requirements on behalf of the single-page application. 

A special handle StatelessAuthHandler will be configured in the request/response chain, and it will handle the OAuth 2.0 authorization code flow for security. Also, the same handler will address the CSRF attacks with CSRF header comparison with JWT token custom claims. 

