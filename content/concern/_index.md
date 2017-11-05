---
title: "Technical Concerns"
date: 2017-11-04T22:04:22-04:00
description: "Plugins or Middleware Handlers that address all the cross-cutting concerns"
categories: [concern]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 01
weight: 01	#rem
aliases: []
toc: false
draft: false
---

One of the design goal of the light-4j is to address all the technical cross-cutting
concerns in the framework so that service developers will only focus on the business
logic without worry about security, auditing, logging, metrics etc. The same pattern
can be applied in the business context as well so that you can have several handlers
to perform different tasks within the business context. For example, the fine-grained
authorization can be done as a cross-cutting concerns in the business context without
mixing into the real business logic within the core request handler. 
