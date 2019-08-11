---
title: "Light Codegen Reference"
date: 2019-01-05T10:47:59-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "reference"
    weight: 80
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

### Why this generator

In the earlier days of light-platform, we only have a Restful framework, and we are leveraging [swagger-codegen][] to generate our projects from Swagger 2.0 specifications. However, there are several issues that we cannot resolve. 

- swagger-codegen does not support Java 8, so we have to fork it and customize it.
- swagger-codegen does not give your flexibility to customize the project. For example, if Oracle Database is supported.
- swagger-codegen can only support the swagger specification, and we later added GraphQL and Hybrid frameworks
- swagger-codegen is very slow, and it takes several seconds to generate a project. Light-codegen needs several milli-seconds so that we can support code generation from the web.
- swagger-codegen does not support external extension, means you have to add your generator to the codebase to work. 
 
Given all the drawbacks, we have decided to build our generator. light-codegen is built with Java 8/11 and using rocker template engine which compiles the templates to Java class to speed it up. It also uses our IoC service module to enable new generators to be added without touch the source code. 

- [Specification Best Practices](/reference/light-codegen/best-practice/)
- [Download or build light-codegen](/reference/light-codegen/download-build/)
- [Swagger 2.0 Generator](/reference/light-codegen/swagger-generator/)
- [OpenAPI 3.0 Generator](/reference/light-codegen/openapi-generator/)
- [OpenAPI 3.0 Kotlin Generator](/reference/light-codegen/openapi-kotlin-generator/)
- [GraphQL Generator](/reference/light-codegen/graphql-generator/)
- [Hybrid Generator](/reference/light-codegen/hybrid-generator/)
- [Eventuate Generator](/reference/light-codegen/eventuate-generator/)
- [DevOps Integration](/reference/light-codegen/integration/)
- [Customization](/reference/light-codegen/customization/)
- [Rocker Hot Reloading](/reference/light-codegen/rocker-hot-reloading/)
- [Different Dockerfile](/reference/light-codegen/dockerfile/)
- [Tutorials](/tutorial/generator/)


[swagger-codegen]: https://github.com/swagger-api/swagger-codegen
