---
title: "Business Handler"
date: 2020-08-30T22:31:11-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The business handler is the last handler in the request/response chain after all the [middleware handlers][]. There are two types of middleware handlers: Technical and Business. 

The Light-Platform is dedicated to providing all the technical middleware handlers to address technical [cross-cutting concerns][] so that users can focus on their business logic to increase productivity and maintainability.

Big organizations will leverage the plugin architecture of the light platform to add another layer of business middleware handlers after the technical middleware handlers in the request/response chain. These handlers are industry-specific, and they usually call them together as their API platform. For example, one of the banking customers built their platform on top of light-4j to address Auditing, Fine-Grained Authorization, Result Filtering and Compliance to share these across all the APIs and applications.

For most developers, they don't need to worry about [cross-cutting concerns][] during the development. What they need to do is to focus on the business handlers generated with the light-codegen.

- [Get Request Info][]
- [Send Response]
- [Rest]
  * [Database]
  * [Kafka]
- [Hybrid]
  * [Kafka]
- [GraphQL]


[cross-cutting concerns]: /concern/
[middleware handlers]: /architecture/middleware-handler/
[Get Request Info]: /development/business-handler/get-request/
[Send Response]: /development/business-handler/send-response/
