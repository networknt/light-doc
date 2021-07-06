---
title: "Light Codegen Tool"
date: 2017-11-05T12:34:57-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 8
weight: 8
sections_weight: 8
aliases: []
toc: false
draft: false
reviewed: true
---

The code generator [light-codegen][] could be used to support project scaffolding for all the frameworks provided by Light.

The code generator based on Rocker template engine that can be used as a utility or web service. It is used currently as scaffold services which include [light-rest-4j][], [light-graphql-4j][],  [light-eventuate-4j][] and [light-hybrid-4j][].

### Workflow

Light-codegen generates initialized microservice projects based on user-defined specification and configuration file:

-- Light-rest-4j

  for OpenAPI based generation:  OpenAPI JSON/YAML format specification and configuration file

  for Swagger based generation:  Swagger JSON format specification and configuration file

-- Light-graphql-4j

  GraphQL IDL for service logic and configuration file


-- Light-hybrid-4j

   Hybrid service JSON based RPC schema definition file and configuration file


-- Light-eventuate-4j (rest API service)

   OpenAPI JSON/YAML format specification which includes the specification for both CQRS command side service and query side service and configuration file



![codegen](/images/codegen.png)



For detail introduction, please refer to [light-codegen][]



[light-codegen]: https://github.com/networknt/light-codegen
[light-graphql-4j]: /style/light-graphql-4j/
[light-hybrid-4j]: /style/light-hybrid-4j/
[light-rest-4j]: /style/light-rest-4j/
[light-eventuate-4j]: /style/light-eventuate-4j/
[light-codegen]: /tool/light-codegen/

