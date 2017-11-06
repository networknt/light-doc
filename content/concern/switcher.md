---
title: "Switcher"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [Concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 110
weight: 110	#rem
aliases: []
toc: false
draft: false
---

This module implement a switcher service interface and a local implementation. Switch 
is useful at system runtime to turn on or off some logic or service given certain
conditions. For example, the light-4j server won't stop handling requests but just
switching off during server shutdown process. The service registry will be notified
but in coming requests are still processed until all clients receives notification from 
service registry. 


