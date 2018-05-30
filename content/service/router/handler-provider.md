---
title: "Handler Provider"
date: 2018-05-29T18:40:03-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The RouterHandlerProvider implements HandlerProvider interface in light-4j to wire in the [RouterProxyClient][] to the request/response chain after other request side middleware handlers. It is also responsible for loading and setting the configurations to the ProxyHandler builder. 

The config file router.yml is self-explained and defined in the [router configuration][].

[router configuration]: /service/router/configuration/
[RouterProxyClient]: /service/router/proxy-client/