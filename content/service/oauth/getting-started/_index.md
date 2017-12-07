---
title: "Getting Started"
date: 2017-12-07T11:42:48-05:00
description: "Getting Started with light-oauth2"
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


If you are new to OAuth2, please read this [introduction][] document to get familiar 
with the concept. 

light-oauth2 are designed as microservices based OAuth2 server that have 7 services and 
numeric endpoints to support user login, access token, user registration, service registration, 
client registration and public key certificate distribution. It can support millions users 
and thousands of clients and services with scopes. It should be easily handle 
thousands of concurrent users per instance and each microservice can be scaled individually.

Specifications can be found at https://github.com/networknt/model-config/tree/master/rest

Three databases are supported: Oracle, Mysql and Postgres


## Build the server from source code

Note: as Oracle client is not available in public maven repo, you have to install it manually by
following this [link][] to install it before building the source code.

```
git clone git@github.com:networknt/light-oauth2.git
cd light-oauth2
mvn clean install
```

[introduction]: /service/oauth/introduction/
[link]: https://dimitrisli.wordpress.com/2012/08/09/maven-install-ojdbc6/

