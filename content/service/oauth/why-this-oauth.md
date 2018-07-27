---
title: "Why This OAuth 2.0 Provider"
date: 2018-06-23T22:12:58-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When your organization wants to adopt API economy, you need a way to secure the API access. OAuth 2.0 is a de facto standard in securing APIs these days. You can choose a third-party cloud service, but most likely you want to deploy an instance of OAuth 2.0 provider in-house.  
There are so many products to choose from the market. Some of them are commercial, and some of them are open source. When you are aiming to microservices, your choice is very limited. The light-oauth2 is designed for microservices architecture at enterprise scale. In fact, it is built as microservices on top of light-4j and light-rest-4j frameworks. There are so many distinct attributes that set this product apart from others. 


### Fast and small memory footprint to lower production cost.

The authorize code service can support 60000 users log in and get authorization code redirect in a second and the token service can generate 700 JWT access tokens per second on my laptop. 

It has 7 microservices connected with an in-memory data grid, and each service can be scaled individually or exposed to outside individually. 

It is open source and free for commercial use with end-user friendly Apache 2.0 license. 

### More secure than other implementations

OAuth 2.0 is just a specification, and many details are in the implementations. Our implementation has a lot of extensions and enhancements for additional security. Also, there are some limitations to prevent users from making mistakes. For example, we have added a new client type called "trusted" and only this type of client can issue resource owner password credentials grant type or any custom grant types. We don't use implicit grant type in this implementation as it is not secure. When dealing with an external client, we can register it as "external" client type so that the token issued to it is a by-reference token that has to be exchanged to JWT once requests come into the internal network.

Microservices architecture reduces the attack surface as with only authorization code, security token, and key distribution services need to be exposed to the network. Other services can be locked down and accessed by light-portal only. 

### More deployment options

You can deploy all services or just deploy several services for your use cases. You can only expose token and code services to the Internet through firewall or light-router installed in DMZ, and all others can only be accessed locally for maximum security. You can have several token services or deploy token service as sidecar pattern in each node. You can start more instance of key service on the day that your public key certificate for signature verification is changed and shut down all of them but one the next day. You can take the full advantages of microservices deployment.  Unlike monolithic OAuth 2.0 providers, each microservice has different IP and port number so that it is easy to control access from F5 or firewall rules. 

### Seamlessly integration with Light-4J frameworks

* Built on top of light-4j and light-rest-4j frameworks
* Light-4j Client and Security modules manage most of the interactions with light-oauth2
* Support service on-boarding from light-portal
* Support client on-boarding from light-portal
* Support user management from light-portal if local database user table is used
* services built on top of the light platform can bootstrap from the key distribution service 
* Source code and OpenAPI specifications for all microservices can be accessed on github.com
* Federated OAuth 2.0 providers can exchange keys upon request from services. 
* A middleware handler is provided to dereference the opaque token to JWT token at the perimeter. 

### Easy to integrate with your APIs or services

The OAuth2 services can be started in a docker-compose for your local development and can be managed by Kubernetes on official test and production environment. It exposes RESTful APIs and can be accessed from all languages and applications.  

### Support multiple databases and can be extended easily

Out of the box, it supports MySQL, MariaDB, Postgres, SQL Server, Oracle and H2 for unit tests. Other databases can be easily added with the configuration change in service.yml and a database script.

### Pluggable authentication provider per client

When authorization code grant type is used, depending on the client id and user type, a customized authentication provider might be invoked. The light-oauth2 provides custom users table, LDAP, SPNEGO/Kerberos, GitHub repository for authentication and authorization. New providers can be plugged in with the service module and client registration. The default authentication provider is SPNEGO/Kerberos with Microsoft AD with basic authentication falls back.

### Federated OAuth 2.0 providers

Most OAuth 2.0 providers support only one instance in an organization. The light-oauth2 supports numeric instances in a federation. For example, an external AS for external clients and an internal AS for internal clients. One instance per data center and all of them are active as independent clusters. A payment service company with an AS that trusts all other partner financial institutions' providers and vice versa. 

### Public key certificate distribution

With distributed security verification, JWT signature public key certificates must be distributed to all resource servers. The traditional push approach is not working with microservices architecture and pull approach is adopted. There is a key service with the endpoint to retrieve the public key certificate from microservices during runtime based on the key_id from the JWT Header. Please refer to [key distribution][] for more details. When federated providers are used, resource servers can go to its provider to get keys from other providers. 

### Two tokens to support microservices architecture

Each service in a microservices application needs a subject token which identifies the original caller (the person who logged in the original client or the origin client in B2B case) and a caller token which identifies the immediate caller (might be another microservice). Both tokens are verified with scopes to the API endpoint level. Additional claims in these tokens are used for fine-grained authorization which happens within the business context. 

### Token exchange for high security

Even with two tokens, we can only verify who is the original caller and which client is the immediate caller. For some highly protected service like payment or fund transfer, we need to ensure that the call is routed through some known services. light-oauth2 token service support token exchange and chaining so that a service can verify the entire call stack to make the decision to grant access or not. 

### Service registration for scope calculation

light-oauth2 has a service registration to allow all service to be registered with service id and all endpoints as well as scopes for the endpoint. During client registration, you can link a client to services/endpoints and the scope of the client can be calculated and updated in client table. This avoids developers to pass in scopes when getting access token as there might be hundreds of them for a client that accesses dozens of microservices. 

### All activities can be audited 

The database audit info handler has been wired into all light-oauth2 services to log each activity across services with sensitive info masked. To enable the audit info handler, simply set the "enableAudit" property in the config file for each service (For example oauth_client.yml for light-oauth2 client service) to true. In the future, we will put these logs into AI stream processing to identify abnormal behaviors just like normal service log processing.  

### OAuth2 server, portal and light frameworks form an ecosystem

[light-4j][] and related frameworks to build API

[light-oauth2][] to control API access and security

[light-portal][] to manage consumers and providers life cycles. 

[light-4j]: /style/
[light-oauth2]: /service/oauth/
[light-portal]: /service/portal/


