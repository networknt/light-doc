---
title: "Subject Token"
date: 2018-03-01T21:37:35-05:00
description: ""
categories: []
keywords: []
weight: 50
aliases: []
toc: false
draft: false
---

As described in [security architecture][], light platform uses two tokens to secure service
to service calls. One token is the original token from the login user and it has original
caller information and possible additional info for fine-grained authorization. This token
is called Subject Token.

The Subject Token represents a person and it has user_id and potential additional user info.

There are two way to get this token. 

* Authorization Grant Flow for OAuth 2.0 provider

* OpenID Connect

This token is passed in request header "Authorization" in most of the cases and if the token
contains scope info to access the immediate API/service, then no Access Token is needed. This
is the situation webserver as a client calling APIs.


[security architecture]: /architecture/security/
[Access Token]: /consumer/access-token/

