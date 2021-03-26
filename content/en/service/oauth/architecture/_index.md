---
title: "Architecture"
date: 2017-12-07T11:40:18-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Unlike web services where all services are built into a monolithic application and we can put a gateway in front of the application for security, the service to service communication adds a lot of overhead if we go through a gateway. We also cannot go to the OAuth 2.0 provider to get token or verify token for each request due to the dramatically increased volume.

The following are some of the architectural considerations for the microservice OAuth 2.0 provider. 
 

### Microservices

It is built on top of light-4j/light-rest-4j frameworks as 7 microservices and each service has several endpoints to support user login, access token retrieval, user registration, service registration, client registration and public key certificate distribution. It can support millions of users and thousands of clients and services with scopes. It should easily handle thousands of concurrent users per instance and each service can be scaled individually if necessary.

### In-Memory Data Grid

Hazelcast is used as a Data Grid across multiple services and the majority of operations won't hit database server for best performance. This also makes the database as a plugin so that persistence layer can be anything from SQL to NoSQL.

### Built-in Security

Except for code, token and key services, other services are protected by OAuth 2.0 itself and additional security as well. These services can be deployed at different locations within your network for maximum security and flexibility. They are optional services and not part of the OAuth 2.0 specification. 

### Multiple Database Support

Currently, Oracle, MySQL, MariaDB, SQL Server and Postgres are supported, but other databases(SQL or NoSQL) can be easily supported by implementing a MapStore of Hazelcast and create an initial database script. 

### Easy to customize and integrate

Each service can be easily customized and won't impact other services. Also, it is very easy to extend in order to integrate with other existing services within your organization. All supported authentication/authorization providers are grouped in the authhub module and you can specify your customized authenticator during client registration. 

### Easy to add new service

Cloud native application security is new and changing on the daily basis. If you need new security features, you can update one of the existing services to add several new endpoints. Or you can add a brand new service if the function is not related to existing services and big enough.

### Cloud and Docker friendly

Designed as native cloud services on top of lightweight Java framework to lower the cost of VM provisioning.

### Federated OAuth 2.0 providers

It can be deployed as multiple clusters. Each cluster works independently and can be registered to another cluster as a trusted provider. Any provider can access trusted providers' public key certificates and make them available for its resource server to verify JWT tokens issued by a trusted provider. 

### By-value and by-reference tokens

It uses by-value tokens inside the corporate network and by-reference token on the Internet. An endpoint is provided to dereference opaque token to JWT once the requests come inside the network. You should set up an external AS and an internal AS to be responsible for external clients and internal clients in this use case. 