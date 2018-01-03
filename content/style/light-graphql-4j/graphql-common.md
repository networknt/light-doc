---
title: "Graphql Common"
date: 2017-11-29T20:59:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This module contains common utilities and controls the configuration for GraphQL service 
and share some static variables with other modules to make the dependencies much simpler.

The major part of this module is GraphqlConfig and it defines the path for the graphql
endpoint and if the GraphiQL is enabled or not. 

Another part of this module is a utility class that is shared within all graphql modules.


Here is an example of graphql.yml

```
# GraphQL configuration
---
# The path of GraphQL endpoint for both GET and POST
path: /graphql

# Enable GraphiQL for development environment only. It will allow you to test from your Browser.
enableGraphiQL: true
```