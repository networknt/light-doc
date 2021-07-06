---
title: "Light Oauth2"
date: 2017-11-05T12:34:02-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 50
weight: 50
sections_weight: 50
aliases: []
toc: false
draft: false
reviewed: true
---

The nature of the services is easy to access so security is the most important thing when deploying services. Light is a security first design and an OAuth 2.0 service provider is available as part of the ecosystem. Along with light-portal client registration and service registration, light-oauth2 provides more than normal OAuth 2.0 for microservices such as public key distribution, scope management, and distributed policy management.

If you are new to OAuth 2.0, please read this [introduction][] document to get familiar with the concept. 

The light-oauth2 is designed as microservices based OAuth 2.0 specifications, and it has 7 services and numeric endpoints to support user login, access token, user registration, service registration, client registration and public key certificate distribution. It can support millions of users and thousands of clients and services with scopes. It should be easily handling thousands of concurrent users per instance and each microservice can be scaled individually.

Specifications can be found at https://github.com/networknt/model-config/tree/master/rest

Five databases are supported: Oracle, Mysql, MariaDB, Postgres and SQL Server out of the box and more will be supported upon customer's request.


### Start light-oauth2 with docker-compose

If you are an API developer that wants to use light-oauth2 to secure your API on your laptop or desktop, you can start the light-oauth2 services with docker-compose. The steps are documented in the [light-oauth2 tutorial][]


### Build the server from source code

If you are a developer that wants to contribute light-oauth2 or debug into it, then you can work on the source code directly. 

Note: as Oracle client is not available in public maven repo, you have to install it manually by following this [link][] to install it before building the source code.

```
git clone https://github.com/networknt/light-oauth2.git
cd light-oauth2
mvn clean install
```

There are docker-compose files available under the root directory of light-oauth2 to build and start all services together. 

Given that the above steps have been completed and each service has a fat jar inside target folder. We can run the following commands to start all services with a different database.

Mysql

```
docker-compose -f docker-compose-mysql.yml up
```

Postgres

```
docker-compose -f docker-compose-postgres.yml up
```

Oracle

```
docker-compose -f docker-compose-oracle.yml up
```

SQL Server

```
docker-compose -f docker-compose-sqlserver.yml up
```

If you change the source code, you have to rebuild it and create docker images again. Remember to remove the existing docker images before running docker-compose again. 

### Tutorials

Once the server is up and running, you can explore them by following the [light-oauth2 tutorial][] step by step. 

### Document

If you need more information please visit [oauth] section in infrastructure service. 


[introduction]: /service/oauth/introduction/
[link]: https://dimitrisli.wordpress.com/2012/08/09/maven-install-ojdbc6/
[light-oauth2 tutorial]: /tutorial/oauth/
[oauth]: /service/oauth/
