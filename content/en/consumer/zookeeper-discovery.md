---
title: "Zookeeper Discovery"
date: 2018-03-01T21:40:40-05:00
description: ""
categories: []
keywords: []
weight: 200
aliases: []
toc: false
draft: false
---

Apache Zookeeper can used as service registry and discovery with [zookeeper][] module in light-4j; however, it is not recommended given it is very heavy and slow. It should only be used when you have the cluster implemented in house already and the number of services running is small. Otherwise, it is recommended to use [Consul][] for service registry and discovery. 

[zookeeper]: /concern/zookeeper/
[Consul]: /consumer/consul-discovery/
