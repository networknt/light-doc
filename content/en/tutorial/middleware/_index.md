---
title: "Middleware Handlers"
date: 2017-12-13T20:41:21-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

The Light platform is a plugin architecture, and there are a lot of middleware handlers provided to address all sort of cross-cutting concerns when you build a microservices. Also, users can add their own plugins within the business context. Basically, the light-4j framework is an embedded gateway that allows users to write their business logic on top of it. Before the request reaches the business handler, there are a lot of middleware handlers need to be executed to address security, metrics, audit, traceability, etc.

In this section, we will include several tutorials that relate to how to use certain middleware handlers.


* [CORS - Cross-Origin Resource Sharing](/tutorial/middleware/cors/)
* [Prometheus for metrics](/tutorial/middleware/prometheus/)


