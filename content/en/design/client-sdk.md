---
title: "Client SDK"
date: 2021-11-09T14:12:15-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Project teams often ask if we can add a codegen feature to generate client SDK to help consumers invoke the API based on the specification. The SwaggerHub codegen does it, and many users are getting used to the client SDK approach provided by the API. 

In my opinion, I don't think it is right to use client SDK to invoke APIs unless the API is public and a lot of clients are trying to integrate with it. For business APIs consumed internally, issuing an SDK to consumers will force all consumers to upgrade once the provider is upgraded. Instead of building independent microservices, we tightly coupled the provider and consumer to build a distributed monolithic application. This pattern is even worse than the monolithic application running in a JavaEE container. 


Another reason we don't want to generate client SDK is due to the maintenance effort. We cannot assume that all clients will be implemented in the same language as the service. So we have to generate client SDK with different popular languages. For each language, there might be different HTTP clients, and we need to consider them. 

If we ever want to help consumers, we would rather create a common library for each language to manage the connection and construct the HTTP request. In this way, we can eliminate the repeatable work from the developers and still give developers the flexibility to invoke APIs based on their business requirements. 

The above discussion is based on the implementation of SwaggerHub client SDK. However, other aspects need to be considered for API consumption. From the client's point of view, some cross-cutting concerns need to be addressed before calling the downstream APIs. For example, how to get a JWT token from the OAuth 2.0 provider and cache it and renew the token without impacting the current request. In a cloud with microservices, how to perform the service discovery before invoking the downstream API etc.


We have provided http-sidecar to help API to API invocation in a Kubernetes cluster for outgoing traffic to answer these questions. 

For standalone consumers, we have provided light-router that can be used along with the consumer to bring the legacy client into the light-4j ecosystem. For more information, please refer to [legacy application](/design/legacy-application/) document. 


