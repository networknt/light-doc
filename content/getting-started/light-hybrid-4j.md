---
title: "Light Hybrid 4j"
date: 2017-11-05T11:46:42-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 30
weight: 30
sections_weight: 30
aliases: []
toc: false
draft: false
---

For public APIs, it is make sense to use RESTful, however, if it is an internal API,
some sort of RPC based API style will be more efficient. As these days, Javascript on
the browser are really powerful and it can deal with JSON or other binary protocol
very easily. These browser object (JSON/Binary) contains type information so that there
is no need to do Object to URI and URI to Object transformation based on OpenAPI spec.
or RAML. 