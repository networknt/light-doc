---
title: "Router Components"
date: 2018-05-29T18:05:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Light-router uses some of the middleware handles from light-4j and light-rest-4j as plugins in the request and response chain. Besides, we have implemented several middleware handlers for light-router specific requirements. These handlers are located in the light-router repository. Some of them are mandatory, and some of them are optional. 

* [RouterProxyClient](/service/router/proxy-client/)
* [RouterHandlerProvider](/service/router/handler-provider/)
* [TokenHandler](/service/router/token-handler/)
* [PathServiceHandler](/service/router/path-service/)
* [PathPrefixServiceHandler](/service/router/path-prefix-serivce/)

