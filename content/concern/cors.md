---
title: "CORS"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
---

If you API server serves SPA (single page application) built on top of Angular
or React, there is no issue for the SPA to access APIs on the same server.
However, some of the single page applications are served by another server
on another domain. In this case, the API server has to handle the pre-flight
options request in order to allow browser client to access the APIs directly. 


## CorsHttpHandler

This handler handles the HTTP pre-flight option request and returns the correct
header to the client. The information returns to the client is controlled by
cors.yml configuration file.

## Configuration

Here is the default configuration cors.yml

```
description: Cors Http Handler
# If cors handler is enabled or not
enabled: true
# Allowed origins, you can have multiple and with port if port is not 80 or 443
# Wildcard is not supported for security reasons.
allowedOrigins:
- http://localhost
# Allowed methods list.
allowedMethods:
- GET
- POST

```

## Tutorial

There is a [tutorial][] that shows you how to use CorsHttpHandler. 

[tutorial]: /tutorial/cors/
