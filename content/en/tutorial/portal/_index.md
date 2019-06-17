---
title: "Light Portal Tutorial"
date: 2018-03-03T20:07:43-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
---

When light-portal is deployed, there might not be a lot of volume in the beginning. We will be using only one hybrid-command server to host all the services that update the portal and one hybrid-query server to host all the services that only read info from the portal. Each server can be scaled to multiple instances based on the read-only load or write-only load. Also, we have an option to move one or two hot services to a separate server instance and scale it accordingly later on.

Most services for portal can be found in the [light-portal][] repository on github.com/networknt. The UI can be found in [view][] folder and the UI final version is deployed to the [light-config-test][] repository on github.com/networknt.


### Build and Start Portal Service

* [light-bot portal build][] - Build portal services with light-bot
* [start portal service][] - Start both command side and query side portal services
* [start local light-router][] - Start light-router instance for service discovery and UI.

### Portal Services:

* [user-management][] - User Management Service

### Portal View 

* [portal view][] - React Portal View


[light-portal]: https://github.com/networknt/light-portal
[user-management]: /tutorial/portal/user-management/
[light-bot portal build]: /tutorial/bot/light-portal-local/
[start portal service]: /tutorial/portal/start-portal-service/
[view]: https://github.com/networknt/light-portal/tree/master/view
[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-router/light-portal/lightapi
[portal view]: /tutorial/portal/view/
[start local light-router]: /tutorial/portal/local-router/
