---
title: "Services"
date: 2017-11-10T14:01:19-05:00
description: "Services provided by light-oauth2"
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

OAuth2 server has a list of services that cover standard OAuth2 grant flows and extended 
features like service on-boarding, client on-boarding, user management, token exchange,
token chaining, scope calculation and public key certificate distribution.  
 
This document only describes the features and processes of each service. Please refer 
to [tutorial][] on how to access these services. 

* [Authorization Code][] - Login to OAuth2 and get authorization code
* [Token Endpoint][] - Token endpoint of OAuth2 provider
* [Service Registration][] - Service registration endpoints
* [Client Registration][] - Client registration endpoints
* [User Management][] - User management endpoints
* [Key Distribution][] - Public key certificate distribution
* [Refresh Token][] - Refresh token service
* [Custom grant type][] - Client authenticated user grant type
* [PKCE][] PKCE implementation
* [OpenID Connect][] OpenID Connect implementation


[OpenID Connect]: /service/oauth/serivce/openid/
[PKCE]: /service/oauth/service/pkce/
[Custom grant type]: /service/oauth/serivce/custom/
[tutorial]: /tutorial/oauth/
[Authorization Code]: /service/oauth/service/code/
[Token Endpoint]: /service/oauth/service/token/
[Service Registration]: /service/oauth/service/service/
[Client Registration]: /service/oauth/service/client/
[User Management]: /service/oauth/service/user/
[Key Distribution]: /service/oauth/service/key/
[Refresh Token]: /service/oauth/service/fresh-token/
