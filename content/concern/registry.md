---
title: "Registry Discovery"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 85
weight: 85	#rem
aliases: []
toc: false
draft: false
---


This module contains all the interfaces that are needed in registry and
discovery. Also it implemented Direct registry which you can hard-code
services into the service.yml in order to simulate consul or zookeeper
during development.

Currently, Consul and ZooKeeper are supported for external service registry
and discovery. 

