---
title: "Rpc Router"
date: 2018-05-07T21:13:30-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The rpc-router is a middleware handler that implements light-4j [handler interface][] and will be injected into the request/response chain. It is responsible for parsing the incoming request body and then routing the request to the right handler to handle it. Several handlers will be packaged into one of the service jar files included into a server instance as a multi-tenancy container.

[handler interface]: /concern/handler/

