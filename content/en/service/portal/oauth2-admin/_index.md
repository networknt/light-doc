---
title: "OAuth Admin"
date: 2020-07-13T15:33:36-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

As you know, we have eight microservices in the light-oauth2, and they are all responsible for different things. 

Services like code, token and key will be accessed from the microservices to obtain authorization code, JWT token and public key certificate. 

Others like client, service and refresh-token are used for OAuth administrators to register clients, services and manipulate refresh tokens. Until now, most users are using curl or postman to access these services through the REST API, which is very difficult to use. 

It has been our priority to add the OAuth 2.0 admin to the light-portal UI and deploy the light-oauth2 services to the cloud so that developers can leverage it without starting light-oauth2 services locally. 

There are three functions on the portal UI to complete the following tasks.

[Client Registration][]
[Service Registration][]
[Refresh Token Admin][]
[Create Test Token][]



[Client Registration]: /service/portal/oauth2-admin/client/
