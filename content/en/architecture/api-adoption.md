---
title: "Stages of API Adoption"
date: 2018-03-15T05:42:50-04:00
description: ""
categories: []
keywords: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

In today’s IT strategy and architecture, APIs are the new frontier. No matter what industry you are in, doing business digitally is the way to go and APIs are an essential part of the digital transformation. As an API platform provider, we have worked with many companies that are building APIs and implementing API management. All of them seem to be traveling a similar path as the kinds of challenges they are facing are almost the same.

From these cases, we have so far categorized five distinct stages of API adoption. Different people might have different
criteria but the overall course is very similar. We just want to share our experience and hopefully help you lead your
API initiative in the right direction. 

### Stage 1: API Awareness - Monolith 

In the first stage of API adoption, APIs get simply introduced as a lightweight method of application integration. Often, we see integration needs of mobile applications as the first targets. In a mobile-first approach, APIs are simply served from existing monolithic applications. For example, existing Java EE applications expose RESTful API through JAX-RS. In other words, APIs at this stage are designed to make life simple for app developers and driven by developers. It greatly increases the development efficiency and significantly speeds-up the overall process, which in turn makes product owners happy. And so, the infections quickly spread.

As APIs are purely driven by mobile applications or single page applications at this stage, there is no active architecture
and design but just modify the traditional web application to expose API in order to speed up the UI development. As the
monolithic platform does not have any built-in features to support API, all cross-cutting concerns will be addressed at
the network level with a commercial API gateway.  

### Stage 2: API Platform Architecture - Modularized Monolith

Once the benefits of APIs are firmly established, you are likely to start thinking about different kinds of concerns and turn to vendors for their API technologies. With more updates to the APIs driven by the business, you start to realize that simply modifying the existing application cannot keep up to speed as deploying a monolithic app is painful and risky. 

You have heard about microservices architecture but cannot go that way as the initial infrastructure cost is too high and building distributed applications is too complicated for the team without experience. Naturally, you would want to separate your APIs to smaller modules and deploy them to the same application server with multiple wars or ears in the Java EE platform. This gives you the flexibility to develop and deploy individual modules independently to meet the ever-changing business requirement.

More and more commercial API management tools might be adopted at this stage as more concerns need to be addressed. For example, security, rate limiting, policy management, etc. need to be considered when more and more APIs are exposed.

### Stage 3: API Management - Entry Level Microservices

The more successful your APIs, the more users for your APIs and gradually the monolith API cannot scale to handle the
volume anymore. At this stage, you are seriously considering microservices architecture and start to redesign some of the
high load APIs to microservices. These microservices are independently developed, deployed, scaled and only focus on
a single functionality. They might still share the same database between different microservices and existing monolith.

From API management perspective, you might start thinking to have centralized logging, auditing, and metrics. You might
consider to dockerize these APIs deploy them into the cloud orchestrated by Kubernetes. 

### Stage 4: Advanced API Management - Medium Level Microservices 

When you have more and more APIs running, you start to realize that API management becomes a daunting task. A shared database tightly couples several APIs together and hinders developers’ abilities to make changes. The single database is broken up and each service has its own database.

API management and infrastructure services become mature and effective. Security, logging, metrics, auditing are all
addressed and most services are running in Kubernetes cluster. You are starting to explore another style of APIs like GraphQL
or RPC.  

### Stage 5: API First - High-Level Microservices

When there are too many updated microservices running within the organization, you soon realize that the synchronous request/response would not work all the time and your application might be broken due to an unreliable network and leave data inconsistent between services. You start to investigate eventual consistency and start to adopt the event-driven asynchronous approach to manage communication between services. Event Sourcing and CQRS will be used in certain applications and Saga pattern will be used to manage distributed transactions between multiple update services. At this stage, service to service communication is mainly done through events and client to service interaction can be RESTful, GraphQL and RPC as well as message, command or event. 

API management is done through a customized portal for the life cycle of APIs and API marketplace will link the consumers to providers. The entire API ecosystem is running organically and more and more APIs add to the system. New application development becomes very easy as most building blocks are available to be reused. 

### Final note

The API era is still fresh and developing rapidly. Consequently, we are far from certain that the fifth stage will be the final stage. Some people say that the end stage will be serverless but nobody knows what is going to happen in a couple of years.

Where are you currently on your API adoption journey? Still in the early stages, or running in the front? Maybe already beyond the fifth stage? No matter what stage you are in, Light provides frameworks, infrastructure services, and toolchains to help you with your API journey.

