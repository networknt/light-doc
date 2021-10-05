---
title: "Decomposition Patterns"
date: 2019-01-25T17:41:41-05:00
description: ""
categories: []
keywords: []
slug: ""
menu:
  docs:
    parent: "design"
    weight: 52
weight: 52
aliases: []
toc: false
draft: false
reviewed: true
---

### Decomposition Patterns

We are constantly asked by our customers on how to decompose legacy monolithic applications to microservices.

##### Decompose by Business Capability

* Problem: Microservices is all about making services loosely coupled and applying the single responsibility principle. However, breaking an application into smaller pieces has to be done logically. How do we decompose an application into small services?

* Solution: One strategy is to decompose by business capability. A business capability is something that a business does in order to generate value. The set of capabilities for a given business depend on the type of business. For example, the capabilities of an insurance company typically include sales, marketing, underwriting, claims processing, billing, compliance, etc. Each business capability can be thought of as a service, except it’s business-oriented rather than technical.

##### Decompose by Subdomain

* Problem: Decomposing an application using business capabilities might be a good start, but you will come across so-called "God Classes" which will not be easy to decompose. These classes will be common among multiple services. For example, the Order class will be used in Order Management, Order Taking, Order Delivery, etc. How do we decompose them?

* Solution: For the “God Classes” issue, DDD (Domain-Driven Design) comes to the rescue. It uses subdomains and bounded context concepts to solve this problem. DDD breaks the whole domain model created for the enterprise into subdomains. Each subdomain will have a model, and the scope of that model will be called the bounded context. Each microservice will be developed around the bounded context.

Note: Identifying subdomains is not an easy task. It requires an understanding of the business. Like business capabilities, subdomains are identified by analyzing the business and its organizational structure and identifying the different areas of expertise.

##### Strangler Pattern 

* Problem: So far, the design patterns we talked about were decomposing applications for Greenfield, but 80% of the work we do is with Brownfield applications, which are big, monolithic applications. Applying all the above design patterns to them will be difficult because breaking them into smaller pieces at the same time they are being used live is a big task.

* Solution: The Strangler pattern comes to the rescue. The Strangler pattern is based on an analogy to a vine that strangles a tree that it’s wrapped around. This solution works well with web applications, where a call goes back and forth, and for each URI call, a service can be broken into different domains and hosted as separate services. The idea is to do it one domain at a time. This creates two separate applications that live side by side in the same URI space. Eventually, the newly refactored application “strangles” or replaces the original application until finally you can shut off the monolithic application.

##### Event Pattern

