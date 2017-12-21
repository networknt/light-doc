---
title: "Distributed session on IMDG"
date: 2017-12-20T21:34:58-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
---

Most our customers build API or microservices to serve Single Page Application on Browser or Native
applications on devices. For security reason, we normally don't send JWT tokens to Browser unless
JWE is applied. In order to secure SPA, session will be used between Browser and [Web Server][]. For
high availability, we need to have multiple instances of web servers behind reverse proxy (F5 etc.)
and sessions need to be replicated between all instances. 



[Web Server]: /style/webserver/
