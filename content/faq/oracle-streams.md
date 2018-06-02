---
title: "Oracle Streams"
date: 2018-06-01T21:38:09-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

We were asked from one of our customers who has unlimited Oracle license if our messaging based framework can utilize Oracle Streams instead of GoldenGate which requires a separate license. 

After several days of investigation, it we don't think Oracle Streams is working in this use case. 

The Oracle Steams is not fit our CDC solution. It can capture the data change to the queue, but the queue is oracle database queue. And if we want to use Oracle Message Gateway. Messaging Gateway enables communication between applications based on non-Oracle messaging systems and Oracle Streams AQ.

But the Oracle Message Gateway currently only supports the integration of Oracle Streams AQ with applications based on WebSphere MQ 6.0 and TIB/Rendezvous 7.2.
I will go through more document and try to prepare a document about it for you
by this weekend

If we use non-oracle message broker, we need  Oracle Message Gateway

it doesn't support kafka for now

