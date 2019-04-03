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

Light-router is primarily used for service discovery and technically there is client-side discovery only as it is called "service discovery" and only clients need to do that. All discovery can only exist on client-side. 

The difference is that the discovery code in client or on the client host or on another static server in a data center. An additional scenario is to use light-router as BFF for SPA or Mobile. 

In scenario one that the code is inside of client which mean using light-4j [client][] module give us most flexibility and performance. 

### Internal

In scenario two that we can deploy light-router on the same host of the client application and it is owned by the client team. In this case, an extra network hop is saved and performance won't be suffered from too much network latency. Another benefit to using this approach is that you can config light-router to enable JWT token renewal and management so that your client code will be transparent
to OAuth 2.0 provider. 

In scenario three that light-router is deployed to a static server in a data center and owned by service provider group or third party common shared service. It is not the recommended approach as there are two network hops and the client cannot trust the light-router instance to handle the security token communication with OAuth 2.0 provider. The performance would be worse than the second option.

### External
  
When using light-router as BFF (backend for frontend) to serve Single Page App or Native Mobile App, it is deployed inside the DMZ and load balanced by F5. It is acting as an embedded API gateway for the client application that connects to it. It adds a layer of security protection, service discovery, client authentication/authorization, and other cross-cutting concerns. 
 

[client]: /concern/client/
