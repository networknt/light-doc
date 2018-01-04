---
title: "Graphql Validator"
date: 2017-11-29T20:59:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Graphql Validator validates the path and methods of the request. Other schema validation will be handled 
by the GraphQL component.

Basic request validation for the graphql path and methods is done by this middleware handler. It is the 
first line of validation right after graphql-security and it doesn’t have any knowledge about the graphql 
query parameter and body. Other schema based validation will be done at GraphQL level.

It shares the same configuration file with swagger-validator or openapi-validator and here is an example.

```
# Validate request/response for GraphQL request
enabled: true
```
