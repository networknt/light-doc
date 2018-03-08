---
title: "Architecture"
date: 2017-12-07T11:40:18-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 20
weight: 20
aliases: []
toc: false
draft: false
---


Unlike web services that all services are built into a monolithic application and we
can put a gateway in front of the application for security, the service to service
communication adds a lot of overhead if we go through gateway. We also cannot go to
the OAuth 2.0 provider to get token or verify token for each request due to the 
dramatically increased volume.

The following are some of the architecture considerations for the microservice OAuth
2 provider. 
 

### Microservices

It is built on top of light-4j/light-rest-4j frameworks as 7 microservices and each service 
has several endpoints to support user login, access token retrieval, user registration, service 
registration, client registration and public key certificate distribution. It can support 
millions users and thousands of clients and services with scopes. It should be easily handle 
thousands of concurrent users per instance and each service can be scaled individually if 
necessary.

### In-Memory Data Grid

Hazelcast is used as Data Grid across multiple services and majority of operations
won't hit database server for best performance. This also makes database as plugin
so that persistence layer can be anything from SQL to NoSQL.


### Built-in Security

Except code, token and key services, other services are protected by OAuth2 itself and 
additional security as well. These services can be deployed at different locations within 
your network for maximum security and flexibility. They are optional services and not
part of the OAuth 2.0 specification.

### Multiple Database Support

Currently, Oralce, MySQL and Postgres are supported, but other databases(sql or nosql) 
can be easily supported by implementing a MapStore of Hazelcast and create a initial 
db script. 

### Easy to customize and integrate

Each service can be easily customized and won't impact other services. Also, it is very 
easy to extend in order to integrate with other existing services within your organization.

### Easy to add new service

Cloud native application security is new and changing on the daily basis. If you need new 
security features, you can update one of the existing service to add several new endpoints.
Or you can add a brand new service if the function is not related to existing services and
big enough.

### Cloud and Docker friendly

Designed as native cloud services on top of light weight Java framework to lower the cost of
VM provisioning.

