---
title: "Light OAuth2"
date: 2017-11-10T12:57:03-05:00
description: ""
categories: [service]
keywords: [oauth2]
menu:
  docs:
    parent: "service"
    weight: 30
weight: 30
aliases: []
toc: false
draft: false
---

Light platform follows security first design and we have provided an OAuth 2.0 provider
light-oauth2 which is based on light-4j and light-rest-4j frameworks with 7 microservices.
Some of the services implement the OAuth 2.0 specifications and others implement some
extensions to make OAuth 2.0 more suitable to protect service to service communication, other 
styles of services like GraphQL, RPC and Event Driven, Key management and distribution,
service registration, token scope calculation and token exchange.    

# Why this OAuth 2.0 Authorization Server

### Fast and small memory footprint to lower production cost.

It can support 60000 user login and get authorization code redirect and can generate 
700 JWT access tokens per second on my laptop. 

It has 7 microservices connected with in-memory data grid and each service can be
scaled individually or exposed to individually. 


### More secure than other implementations

OAuth 2.0 is just a specification and a lot of details are in the individual
implementation. Our implementation has a lot of extensions and enhancements 
for additional security and prevent users making mistakes. For example, we
have added an additional client type called "trusted" and only this type of
client can issue resource owner password credentials grant type or custom grant
types. 

### More deployment options

You can deploy all services or just deploy several services for your use cases. You 
can only expose token and code services to the Internet through firewall or light-proxy
installed in DMZ and all others only can be accessed locally for maximum security.
You can have several token services or deploy token service as sidecar pattern in
each node. You can start more instance of key service on the day that your public
key certificate for signature verification is changed and shutdown all of them but
one the next day. You can take the full advantages of microservices deployment.  

### Seamlessly integration with Light-4J frameworks

* Built on top of light-4j and light-rest-4j
* Light-4j Client and Security modules manages most of the communications with OAuth
* Support service on-boarding from light-portal
* Support client on-boarding from light-portal
* Support user management from light-portal
* Open sourced OpenAPI specifications for all microserivces

### Easy to integrate with your APIs or services

The OAuth2 services can be started in a docker-compose for your local development and 
can be managed by Kubernetes on official test and production environment. It exposes
RESTful APIs and can be access from all languages and applications.  

### Support multiple databases and can be extended and customized easily

Out of the box, it supports Mysql, Postgres and Oracle XE and H2 for unit tests. Other
databases can be easily added with configuration change in service.yml and a dbscript.


### Public key certificate distribution

With distributed security verification, JWT signature public key certificates must
but distributed to all resource servers. The traditional push approach is not
working with microservices architecture and pull approach is adopted. There is a 
key service with endpoint to retrieve public key certificate from microservices 
during runtime based on the key_id from JWT header. Please refer to [key distribution][]
for more details.  

### Two tokens to support microservices architecture

Each service in a microservices application needs a subject token which identifies the
original caller (the person who logged in the original client) and an access token
which identifies the immediate caller (might be another microservice). Both tokens
will be verified with scopes to the API endpoint level. Additional claims in these
tokens will be used for fine-grained authorization which happens within the business
context. 

### Token exchange for high security

Even with two tokens, we can only verify who is the original calller and which client is
the immediate caller. For some highly protected service like payment or fund transfer,
we need to ensure that the call is routed through some known services. light-oauth2
token service support token exchange and chaining so that a service can verify the
entire call stack to make decison to grant access or not. 

### Service registration for scope calculation

light-oauth2 has a service registration to allow all service to be registered with service
id and all endpoints as well as scopes for the endpoint. During client registration, you
can link a client to services/endpoints and the scope of the client can be calculated
and updated in client table. This avoids developers to pass in scopes when getting
access token as there might be hundreds of them for a client that accesses dozens of
microservices. 

### All activities are audited 

A database audit handler has been wired into all light-oauth2 services to log each
activity across services with sensitive info masked. In the future we will put these
logs into AI stream processing to identify abnormal behaviors just like normal service
log processing.  

### OAuth2 server, portal and light frameworks form an ecosystem

[light-4j][] and related frameworks to build API

[light-oauth2][] to control API access

[light-portal][] to manage consumers and providers life cycles. 


# Introduction

This [introduction][] document contains all the basic concept of OAuth 2.0 specification
and how it work in general. 

# Getting started

The easiest way to start using light-oauth2 in your development environment is through
docker-compose in light-docker repository. Please refer to [getting started][] for more
information. If you don't want to run light-oauth2 locally, you can use our [cloud services][]
for development. 

# Architecture

There are some key decision points that are documented in [architecture][] section.

# Documentation

The detailed [service document][] help users to understand how each individual service
works and the specification for each service. It also contains information on which
scenarios will trigger what kind of errors. 

# Tutorial

There are [tutorials][] for each service that show how to use the most common use cases
with examples. 

# Reference

There are vast amount of information about OAuth 2.0 specifications and implementations. 
Here are some important [references][] that can help you to understand OAuth 2.0 Authorization.


[light-4j]: https://github.com/networknt/light-4j
[light-oauth2]: https://github.com/networknt/light-oauth2
[light-portal]: https://github.com/networknt/light-portal
[light-oauth2 service]: /service/oauth/service/
[light-oauth2 tutorial]: /tutorial/oauth/
[getting started]: /getting-started/light-oauth2/
[architecture]: /service/oauth/architecture/
[service document]: /service/oauth/service/
[tutorials]: /tutorial/oauth/
[references]: /service/oauth/reference/
[introduction]: /service/oauth/introduction/
[key distribution]: /architecture/key-distribution/
[cloud service]: /lightapi.net
