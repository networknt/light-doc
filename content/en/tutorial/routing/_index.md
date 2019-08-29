---
title: "Handler Routing Tutorial"
date: 2018-07-08T17:45:44-05:00
lastmod: 2018-07-08
toc: true
draft: false
reviewed: true
---

When building a microservice, there are many cases where you might need different handlers to run for different request paths that your service supports. For example, the `/health` endpoint might not need schema validation, or the `/info` endpoint may not need to push traceability metrics. 

In any case, we need to provide a way to configure which request paths hit which handler chains. And the following tutorial will guide you through the process of what building out the config you will need to support multi handler chains. 

- [Define Handler Chain with handler.yml](/tutorial/routing/handler/)

The handler.yml configuration file gives you a straightforward way to define multiple paths and handler chains for each. Sometimes, you need to specify the routing to gain the flexibility manually. 

- [Manually route requests to a handler](/tutorial/routing/manual/)
