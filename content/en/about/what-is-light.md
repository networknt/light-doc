---
title: What is Light Platform
linktitle: What is Light
description: Light is a platform written in Java SE and designed to build cloud native Web/API with different options.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
layout: single
menu:
  docs:
    parent: "about"
    weight: 1
weight: 1
sections_weight: 1
draft: false
toc: true
reviewed: true
---

Light-4j is a general-purpose Web/API framework built on top of Java SE. Technically speaking, Light consists of over ten frameworks for different styles of API building with infrastructure services like OAuth2, Portal, Logging, Messaging and Metrics, etc. There is also a list of tools to help increase the productivity of developers and operation staff.

Websites or services built with Light are extremely fast and secure. Light services can be hosted anywhere, including cloud providers or on-premise data centers. To alleviate the pain of small or medium-sized businesses managing their own infrastructure, we are planning to provide all the services hosted at lightapi.net including serverless solutions built on top of light-hybrid-4j and light-eventuate-4j frameworks.

We think of Light Platform as the ideal website or services creation tool with flexible architecture so that plugins can be injected or enabled with externalized configuration changes. 

## How Fast is Light Platform?
Visit our github link to compare how fast Light is:   
https://github.com/networknt/microservices-framework-benchmark

## What Does Light Platform Do?

In technical terms, Light takes an IDL (Interface Definition Language) and scaffolds a new service project with light-codegen for developers to add only business logic in generated handlers and write unit tests for handlers. It also provides a toolchain for building and delivering the dockerized image to the target environment. During runtime, there are light-portals to manage the services and light-oauth2 to secure the services to services interactions.

## Who Should Use Light Platform?

* Light is for people who need to handle huge volumes as it can handle millions of requests per second on commodity hardware.

* Light is for people who want to save money on production costs as one instance of Light can replace dozens or hundreds of other instances built with the Java EE platform.

* Light is for people who want to build a fully distributed system without a central point of bottlenecks and failures.

* Light is for people who want to increase developer productivity and bring their products
to the market ASAP.
