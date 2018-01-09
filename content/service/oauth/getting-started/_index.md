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

light-oauth2 is designed as microservices based OAuth2 server that has 7 services and 
numeric endpoints to support user login, access token, user registration, service registration, 
client registration and public key certificate distribution. It can support millions users 
and thousands of clients and services with scopes. It should be easily handling 
thousands of concurrent users per instance and each microservice can be scaled individually.

Specifications can be found at https://github.com/networknt/model-config/tree/master/rest/swagger

Three databases are supported: Oracle, Mysql and Postgres


### Start light-oauth2 with docker-compose

If you are an API developer that want to use light-oauth2 to security your API on your laptop
or desktop, you can start the light-oauth2 services with docker-compose. The steps are documented
in the [light-oauth2 tutorial][]


### Build the server from source code

If you are a developer that want to contribute light-oauth2 or debug into it, then you can work
on the source code directly. 

Note: as Oracle client is not available in public maven repo, you have to install it manually by
following this [link][] to install it before building the source code.

```
git clone https://github.com/networknt/light-oauth2.git
cd light-oauth2
mvn clean install
```

There are docker-compose files available under the root directory of light-oauth2 to build and 
start all services together. 

Given that the above steps have been completed and each service has a fat jar in side target folder.
we can run the following commands to start all services with different database as repository

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


[introduction]: /service/oauth/introduction/
[link]: https://dimitrisli.wordpress.com/2012/08/09/maven-install-ojdbc6/
[light-oauth2 tutorial]: /tutorial/oauth/start/

