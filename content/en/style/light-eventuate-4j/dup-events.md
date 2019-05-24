---
title: "How to handle duplicated events in Kafka"
date: 2018-01-06T17:05:17-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

As we are using Kafka as our messaging broker for event sourcing and Kafka
guarantees messages are delivered to consumers at least once. that means we
will have duplicated events that have to be handled gracefully. 

There are several ways to handle duplicate messages. 


## Idempotency

## Vector clock

