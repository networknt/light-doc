---
title: "Access Token"
date: 2018-03-01T21:37:48-05:00
description: ""
categories: []
keywords: []
weight: 60
aliases: []
toc: false
draft: false
reviewed: true
---

Access Token is used for service to service invocation or standalone applications which invoke a
service. Usually, this token is retrieved from OAuth 2.0 provider by following the client credentials
grant flow. The token doesn't have any information about a user but only has client_id to indicate
who is the calling application. 

Access Token is very important in carrying scopes to the calling service as [Subject Token][] won't
have the scopes for all the services in the invocation chain. 

If Access token is the only token available, it is passed in Authorization header otherwise, it is
passed in as X-Scope-Token header in the request.


[Subject Token]: /consumer/subject-token/
