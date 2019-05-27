---
title: "Light Codegen Reference"
date: 2019-01-05T10:47:59-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "references"
    weight: 80
slug: ""
aliases: []
toc: false
draft: false
---

### Why this generator

In the ealier days of light-platform, we only have a Restful framework, and we are leveraging [swagger-codegen][] To generate our projects from Swagger 2.0 specifications. However, there are several issues that we cannot resolve. 

- swagger-codegen does not support Java 8, so we have to fork it and customize it.
- swagger-codegen does not give your flexibility to customize the project. For example, if Oracle is supported.
- swagger-codegen can only support swagger specification, and we later added GraphQL and Hybrid frameworks
- swagger-codegen is very slow, and it takes several seconds to generate a project. light-codegen needs several milli-seconds so that we can support code generation from the web.
- swagger-codegen does not support external extension, means you have to add your generator to the codebase to work. 
 
Given all the drawbacks, we have decided to build our own generator. light-codegen is built with Java 8 and using rocker template engine which compiles the templates to Java class to speed it up. It also uses our IoC service module to enable new generators to be added without touch the source code. 

- [Specification Best Practices](/references/light-codegen/best-practice/)
- [Swagger 2.0 Generator](/references/light-codegen/swagger-generator/)
- [OpenAPI 3.0 Generator](/references/light-codegen/openapi-generator/)
- [OpenAPI 3.0 Kotlin Generator](/references/light-codegen/openapi-kotlin-generator/)
- [GraphQL Generator](/references/light-codegen/graphql-generator/)
- [Hybrid Generator](/references/light-codegen/hybrid-generator/)
- [Eventuate Generator](/references/light-codegen/eventuate-generator/)
- [DevOps Integration](/references/light-codegen/integration/)
- [Customization](/references/light-codegen/customization/)
- [Tutorials](/tutorial/generator/)


[swagger-codegen]: https://github.com/swagger-api/swagger-codegen
