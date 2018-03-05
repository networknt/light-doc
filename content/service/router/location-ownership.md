---
title: "Location Ownership"
date: 2018-03-05T10:59:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-router is primary used for service discovery and technically there is only client side
discovery as it is called "service discovery" and only client need to do that.

The difference is that the discovery code in client or on the client host or on another static
server in data center.

In scenario one that the code is inside of client which mean using light-4j [client][] module give
us most flexibility and performance. 

In scenario two that we can deploy light-router on the same host of client application and it is
owned by the client team. In this case, an extra network hop is saved and performance won't be
suffered from too much network latency. Another benefit to use this approach is that you can config
light-router to enable JWT token renewal and management so that you client code will be transparent
to OAuth 2.0 provider. 

In scenario three that light-router is deployed to a static server in data center and owned by 
service provide group or third party common shared service. This is not the recommended approach
as there are two network hops and client cannot trust the light-router instance to handle the
security communication with OAuth 2.0 provider. The performance would be worse than the second
option.  
 

[client]: /concern/client/
