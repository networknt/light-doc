---
title: "Start a service from light-codegen"
date: 2019-03-11T23:06:27-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-4j framework is a [design-driven][] framework which means you have to start with the specification before starting with code. It makes sense as the contract is the most important thing for a service and it should be finalized before coding is started. The easy way to start a microservice with the light-rest-4j framework is to use [light-codegen][] to scaffold a running application based on your Swagger 2.0 specification or OpenAPI 3.0 specification and a config.json that controls how code generator works.

At the moment, we support both the Swagger 2.0 specification and the OpenAPI 3.0 specification. Both specs are not the same, and they are not backward compatible, so we have to create two sets of middleware handlers to address them separately.

If you don't know which specification to use, please follow the OpenAPI 3.0 tutorial as it is the future.

[light-codegen]: /tool/light-codegen/
[design-driven]: /design/design-first/
