---
title: "End-to-End Test"
date: 2017-11-06T17:45:44-05:00
description: "End-to-End Test"
categories: []
keywords: [tutorial]
aliases: []
weight: 80
toc: false
draft: false
---

Integration test in monolithic application is complicated as there might be database that
need to be populated and there might be third party systems that need to be running in order
to test all functionalities. In microservices architecture, it is even worse as there are
more services that need to be running together in order to test a single service. For simple
application which consists only several services, it is OK to be tested on developers laptop
or desktop with docker compose. If the application involves dozens services and most services
have their own databases, then it must be tested in a cloud environment. 

In order to test our own libraries and services, we have developed [light-bot][] and one of
the features is to checkout develop branch for all repositories and build/test them all by
starting all servers together. 


##### Return to [Common Tutorial Home Page](/tutorial/common)


[light-bot]: https://github.com/networknt/light-bot
