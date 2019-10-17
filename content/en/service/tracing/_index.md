---
title: "Tracing"
date: 2019-07-05T10:31:40-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Traditionally, most applications have been developed as monolithic. Monolithics are compiled and packaged into a single deliverable and deployed as a single unit. That single unit contains thousands or even millions of lines of code. This method of software delivery has been eclipsed in the last several years by microservice architecture. An application can consist of dozens or hundreds of microservices which can be developed and deployed independently. Each microservice performs one essential function of the application, and nothing more. The contract/API governs all interactions with microservices. 


The operational differences between monoliths and microservices are different. When troubleshooting a failure, monoliths are convenient because all the code is bundled together. It’s easy to track a request to the code through its entire lifecycle. In a distributed architecture like microservices, the same original request might travel several deployable, and the traditional logging and debugging tools are not suitable.




