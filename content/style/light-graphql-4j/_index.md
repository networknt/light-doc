---
title: "GraphQL - A query language for your API"
date: 2017-11-29T21:00:16-05:00
description: ""
categories: [style]
keywords: [graphql]
menu:
  docs:
    parent: "style"
    weight: 10
weight: 10	#rem
aliases: []
toc: false
draft: false
---

GraphQL is a new style of protocol open sourced by Facebook and it is getting popular 
especially in Mobile and Single Page Application as front end. GraphQL provides a 
complete and understandable description of the data in your API, gives clients the 
power to ask for exactly what they need and nothing more, makes it easier to evolve 
APIs over time, and enables powerful developer tools.

Light-graph-4j is a framework that is designed to work with GraphQL. Similar to other 
Light frameworks it provides components and middleware handlers to handle specific use 
cases.

### GraphQL Specific Components

* [graphql-common][] contains common utilities and static variables that are shared by other components.
* [graphql-router][] is responsible for handling GraphQL and GraphiQL requests and hooks schema provider.

### GraphQL Specific Middleware Handlers

* [graphql-security][] verifies JWT token in request header and verifies scopes if it is enabled.
* [graphql-validator][] validates the path and methods of the request. Other schema validation will be handled by the GraphQL component.


[graphql-common]: /style/graphql-common/
[graphql-router]: /style/graphql-router/
[graphql-security]: /style/graphql-security/
[graphql-validator]: /style/graphql-validator/
