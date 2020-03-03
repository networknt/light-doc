---
title: "Light Portal Tutorial"
date: 2018-03-03T20:07:43-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Light-portal is the central point that provides entries for all light-platform supporting services. It is a website that integrates all services with a standard UI for users to utilize and manage these services. Behind the single-page application is a light-router instance that hosts all the virtual domains and proxy API requests to the backend services. 

Most light-portal services are built on top of the light-hybrid-4j framework with Event Sourcing and CQRS. Some of the portal services like the light-oauth2 are built on top of the light-rest-4j framework. 

When light-portal services are deployed with the light-hybrid-4j, there might not be a lot of volume in the beginning. We use one hybrid-command server to host all the services that update the portal and one hybrid-query server to host all the services that only read info from the portal. Each server can be scaled to multiple instances based on the read-only load or write-only load. Also, we have an option to move one or two hot services to a separate server instance and scale it accordingly later on.

Most services for the portal can be found in the [light-portal][] repository on github.com/networknt. The UI can be found in [view][] folder, and the UI final version is deployed to the [light-config-test][] repository on github.com/networknt.

### Build and Start Portal Service

* [light-bot portal build][] - Build portal services with light-bot
* [start portal service][] - Start both command side and query side portal services
* [start local light-router][] - Start light-router instance for service discovery and UI.

### Portal Services:

* [user-management][] - User Management Service

### Portal View 

* [portal view][] - React Portal View

### Developer

* [React View Developer][]
* [Full Stack Developer][]


[light-portal]: https://github.com/networknt/light-portal
[user-management]: /tutorial/portal/user-management/
[light-bot portal build]: /tutorial/bot/light-portal-local/
[start portal service]: /tutorial/portal/start-portal-service/
[view]: https://github.com/networknt/light-portal/tree/master/view
[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-router/light-portal/lightapi
[portal view]: /tutorial/portal/view/
[start local light-router]: /tutorial/portal/local-router/
[setup view dev environment]: /tutorial/portal/view-dev-env/
[React View Developer]: /tutorial/portal/developer/react-ui/
[Full Stack Developer]: /tutorial/portal/developer/full-stack/
