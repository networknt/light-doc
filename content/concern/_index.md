---
title: "Concerns Overview"
date: 2017-11-04T22:04:22-04:00
description: "Plugins or Middleware Handlers that address all the cross-cutting concerns"
categories: [concerns,fundamentals]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 01
weight: 50	#rem
sections_weight: 50
aliases: []
toc: false
draft: false
---

One of the design goal of the light-4j is to address all the technical cross-cutting
concerns in the frameworks so that service developers will only focus on the business
logic without worry about security, auditing, logging, metrics etc. The same pattern
can be applied in the business context as well so that you can have several handlers
to perform different tasks within the business context. For example, the fine-grained
authorization can be done as a cross-cutting concerns in the business context without
mixing into the real business logic within the core request handler. 

## Technical Cross-Cutting Concerns

When building an application, there is function requirement and normally there is
always a non-functional requirement that is created to address all technical concerns.
In most platforms, developers need to address these concerns in the same business
domain at the same time. This makes the application very hard to maintain and hard
to reason about as function requirement and business requirement are implemented at
the same time in the same context. The code is blended and hard to separate, hence
decrease the maintainability. 


## Business Cross-Cutting Concerns

In a system, beside of technical concerns there are other concerns that need to be
addressed within the business context. In most systems, these will be blended into
the business logic implementation which make the business rules hard to reason about. 

