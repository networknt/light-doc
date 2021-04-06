---
title: "Light Saga 4j"
date: 2017-11-05T12:31:18-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 90
weight: 90
sections_weight: 90
aliases: []
toc: false
draft: false
---

In microservices architecture, one request might trigger multiple services in the call stack, so to ensure data consistency between services, we have provided light-saga-4j to manage distributed transactions. Sagas require your services to be idempotent and provide a compensation action for each update action. light-saga-4j is built on top of light-eventuate-4j.


