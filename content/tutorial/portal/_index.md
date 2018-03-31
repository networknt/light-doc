---
title: "Light Portal Tutorial"
date: 2018-03-03T20:07:43-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 170
weight: 170
aliases: []
toc: false
draft: false
---

When light-portal is deployed, there might not be a lot of volume in the beginning. 
We will be using only one hybrid-command server to host all the services that update
the portal and one hybrid-query server to host all the services that only read info
from the portal. Each server can be scaled to multiple instances based on the read-only 
load or write-only load. Also, we have an option to move one or two hot services to a 
separate server and scale it accordingly later on.

All services can be found in in [light-portal][] repository on github.com

### Build and Start Portal Service

* [light-bot portal build][] - Build portal services with light-bot
* [start portal service][] - Start both command side and query side portal services

### Command Services:

* [user-management][] - User Management Service

### Query Services: 

* [api-certification][] - API-Certification Service

### Portal View 


[light-portal]: https://github.com/networknt/light-portal
[api-certification]: /tutorial/portal/api-certification/
[user-management]: /tutorial/portal/user-management/
[light-bot portal build]: /tutorial/bot/light-portal-local/
[start portal service]: /tutorial/portal/start-portal-service/
