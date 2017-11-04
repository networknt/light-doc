---
title: What is Light Platform
linktitle: What is Light
description: Light is a platform written in Java SE, and designed to build cloud native Web/API with different options.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
layout: single
menu:
  main:
    parent: "about"
    weight: 10
weight: 10
sections_weight: 10
draft: false
aliases: [/overview/introduction/,/about/why-i-built-light/]
toc: true
---

Light-4j is a general-purpose Web/API framework built on top of Java SE. Technically 
speaking, Light Platform consists of several frameworks for different style of API
building with infrastructure services like OAuth2, Portal, Logging, Messaging and Metrics
etc. There are also a list of tools to help increase productivity of developers and
operation staff. 


Websites or services built with Light Platform are extremely fast and secure. Light
service can be hosted anywhere, including cloud providers or on premise data centers.
To alleviate the pain for small or medium size businesses to manage their own 
infrastructure, we are planning to provide all the services hosted at lightapi.net
including serverless solution built on top of light-hybrid-4j and light-eventuate-4j
frameworks. 


We think of Light Platform as the ideal website or services creation tool with flexible
architecture so that plugins can be injected or enabled with externalized configuration 
changes. 

## How Fast is Light Platform?

{{< youtube "CdiDYZ51a2o" >}}

## What Does Light Platform Do?

In technical terms, Light Platform takes an IDL (Interface Definition Language) and 
scaffolds a new service project with light-codegen for developers to add only business 
logic in generated handlers and write unit tests for handlers. It also provide a tool
chain for building and delivery the dockerized image to the target environment. During
runtime, there are light-portal to manage the services and light-oauth2 to secure the
services to services interactions. 

## Who Should Use Light Platform?

* Light is for people who need to handle huge volume as it can handler millions requests
per seconds on commodity hardware. 

* Light is for people who want to save money on production cost as one instance of light
can replace dozens or hundreds other instances built with Java EE platform. 

* Light is for people who want to build a fully distributed system without central point
of bottlenecks and failures. 

* Light is for people who want to increase developer productivity and bring their products
to the market ASAP.

