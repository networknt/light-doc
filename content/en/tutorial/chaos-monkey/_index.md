---
title: "Light Chaos Monkey"
date: 2021-02-01T22:51:37-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For large scaled microservices deployment on the cloud, many unpredictable issues might happen on production. We can always guess the application's behaviour when something happens, but we can never verify the hypothesis. To ensure the system resiliency, Chaos engineering has been introduced, and a commonly used tool is Chaos Monkey. 

As light-4j addresses the cross-cutting concerns with middleware handlers in the request/response chain, we can easily design the Chaos Monkey tools with middleware handlers to injected into the live application with a disabled state by default. By providing an API, testers can dynamically enable the Chaos Monkey handlers at runtime with different configurations to test the system response with average production request volume. 


To show users how to use the light-chaos-monkey handlers and API, we are going to use the petstore API as a base to make configuration changes to demo the features of Chaos Monkey assault handlers.


* [Petstore Chaos Monkey](/tutorial/chaos-monkey/petstore/)

