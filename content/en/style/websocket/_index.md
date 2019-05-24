---
title: "Websocket with light platform"
date: 2017-12-20T21:21:31-05:00
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

Light-4j is built on top of Undertow core and it support Websocket natively without going through
Java EE stack. So the performance is much better than other servers in term of throughput. There
are two examples in [light-example-4j][] to demo client-to-server and peer-to-peer websocket. 

[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/websocket
