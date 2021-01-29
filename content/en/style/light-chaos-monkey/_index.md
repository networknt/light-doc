---
title: "Chaos Monkey"
date: 2021-01-28T12:01:21-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Inspired by Chaos Engineering at Netflix and Chaos Monkey for Spring Boot.

### The Goal of Chaos Monkey

While helping my customers to build large scale distributed systems in light-4j, I constantly receive the following questions:

* Will the fallback work?
* How would the application behave with network latency?
* What will happen if one of the service instances is down?
* How does the service discovery help during service deployment?

As you can see, there are a lot of questions to be answered to make our customers comfortable to rollout distributed applications. 

These questions led me to dive into [Chaos Engineering](https://principlesofchaos.org/) and started this project to share my thoughts and experience.

### How does it work?

As we know, the light-4j framework is using middleware handlers to address cross-cutting concerns in the request/response chain in the same service instance or at the network level with light-router or light-proxy. 

To simulate some of the Chaos Monkey behaviours, we built several middleware handlers that can be injected into the request/response chain. These middleware handlers simulate network latency, out of memory, runtime exception and application shutdown etc. 

### Dynamic Configuration at runtime

Each middleware handler has its configuration file with a switch to enable or disable. So you can even deploy it to the production without any negative impact. 

A configuration endpoint /chaosmonkey enables these handlers at runtime with different parameters. It allows controlled Chaos Engineering from the pipeline so that it can be rerun for every release. Also, this gives the team more opportunities to simulate random production behaviours. 

### Assault Handlers

Assault handlers are the heart of Chaos Monkey and the service makes use of them based on your configuration. Following assault handlers are actual provided.

* [Latency Assault][]

If Latency Assault is enabled, latency is added to a request. You control the number of requests where this should occur via the level.

* [Exception Assault][]

You can determine at runtime whether an exception should occur when the method is used.

* [KillApp Assault][]

When the configured methods are called in the application, the Chaos Monkey will shut down the application.

* [Memory Assault][]

Memory Assaults attack the memory of the Java Virtual Machine.


[Latency Assault]: /style/light-chaos-monkey/latency-assault/
[Exception Assault]: /style/light-chaos-monkey/exception-assault/
[KillApp Assault]: /style/light-chaos-monkey/killapp-assault/
[Memory Assault]: /style/light-chaos-monkey/memory-assault/
