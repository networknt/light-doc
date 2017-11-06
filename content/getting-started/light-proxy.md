---
title: "Light Proxy"
date: 2017-11-05T12:31:31-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 70
weight: 70
sections_weight: 70
aliases: []
toc: false
draft: false
---

For an organization that established infrastructure to support services built on
top of light platform, chances are they have some existing services built with
other platforms or even other languages. In order to leverage the security, metrics,
logging, auditing, client side service discovery etc. for these existing services, it is
a good idea to put [light-proxy][] in front of these services to provide cross-cutting
concerns for them. Although the light-proxy adds an additional network hop, however, it
might be faster given it support HTTP 2.0 by default. 

[light-proxy]: /service/light-proxy/
