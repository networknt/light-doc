---
title: "Why not use kafka as eventstore"
date: 2018-01-06T15:35:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When using Kafka as pipeline for event sourcing, people will ask why not use
Kafka as event store instead of a database. It can persist events and keep it 
for as long as it requires.

There are several drawbacks in doing so.

- Kafka only guarantees at least once delivery and there are duplicates in the event
store that cannot be removed.

- Due to immutability, there is no way to manipulate event store when application
evolves and events need to be transformed.

- No place to persist snapshots of entities/aggregates and replay will become slower
and slower.

- Given Kafka partitions are distributed and they are hard to manage and backup
compare with databases.


Given above disadvantages, a database is used as event store instead in eventuate
architecture. 

