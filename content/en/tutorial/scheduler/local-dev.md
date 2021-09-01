---
title: "Scheduler Local Development"
date: 2021-08-31T18:00:16-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The following are steps to get the development environment ready on your desktop. 


### Confluent Platform

You can start the Confluent Platform with [confluent local][] or [confluent docker][] and most users prefer the later. 



### Create Topics

We need to create at least two topics. One for the task definitions and the other for the schedule messages. By default, we are using the light-scheduler topic name. Create the following two topics with 16 partitions from the [control center](http://localhost:9021).


```
light-scheduler
controller-health-check

```

Here is a video to walk through the development process.

{{< youtube W_ceowazWMU >}}

