---
title: "Hybrid - Modularized Monolithic"
date: 2017-12-20T21:21:09-05:00
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

For most organizations, switch from monolithic architecture to microservices is really hard as the
initial infrastructure investment is very high and almost all the design patterns in Java EE are
anti-patterns in microservices. It might be easier to start with modularized monolithic and then
split each module to separate instances as microservices once volume picks up. light-hybrid-4j is
designed for it. It takes advantages of both monolithic and microservices with simple RPC style of
interaction between services.

