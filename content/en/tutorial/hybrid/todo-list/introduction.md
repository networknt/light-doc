---
title: "Introduction"
date: 2019-04-09T10:30:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

A hybrid service is a type of service between JEE service and microservice. It can help some organizations to take step by step approach to fully adopt microservices. Users can split the original large project to several Hybrid services to run at same JVM. From the consumer side, Hybrid services can be treated as microservices.

The light-eventuate-4j framework is designed for Event Sourcing and CQRS based on the events on Kafka. It is a way to enable communication between services asynchronously and maintain the data consistency between services eventually. 

In this tutorial, we are using light-eventuate-4j’s todo-list example to generate and run hybrid command and query services. That means we are going to build both services in the light-hybrid-4j framework and they communicate with each other through the eventuate framework. 

The first step is to [prepare the workspace][] for our tutorial.

[prepare the workspace]: /tutorial/hybrid/todo-list/workspace/
