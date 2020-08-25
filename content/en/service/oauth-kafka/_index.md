---
title: "OAuth Kafka"
date: 2020-08-21T21:28:43-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

While working on the light-portal to provide subscription services to small and medium-sized customers, we realized that the light-oauth2 is not very suitable for the internet scale.

First, it is based on a SQL database that is not scaling very well. All though we have a cache layer to share data between different microservices, it is not very friendly with Docker and Kubernetes.

Our OAuth 2.0 provider has a lot of extra features to support microservices deployment. When integrating with light-portal, some of those services are covered by the marketplace already. For example, client registration and service registration will be covered in the marketplace for the client and service publishing. The user service will be covered by the user management service in the portal. When all supporting services are removed, the only core services are code, token and key. 

It has been a long time that light-oauth2 can authenticate users with user management services of light-portal by configuring authenticate class during the client registration. With client and service registration available from light-portal, it makes sense to use them instead of database-based implementation in the light-oauth2. 

Light-portal is based on light-kafka for event sourcing and CQRS, and it is highly scalable. So the decision is to create another OAuth 2.0 implementation with light-portal integration. 

The new repository is named oauth-kafka, and it is a private on GitHub.com. The source code is available for developers and enterprise customers. 

