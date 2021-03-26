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
reviewed: true
---

This module contains common utilities, controls the configuration for GraphQL service and shares some static variables with other modules to make the dependencies much simpler.

The significant part of this module is GraphqlConfig, which defines the path for the /graphql endpoint and if GraphiQL is enabled or not.

Another part of this module is a utility class that is shared within all graphql modules.

Here is an example of graphql.yml

```
# GraphQL configuration
---
# The path of GraphQL endpoint for both GET and POST
path: /graphql

# Enable GraphiQL for development environment only. It will allow you to test from your Browser.
enableGraphiQL: true

# Path to the websocket endpoint to handle subscription requests.
subscriptionsPath: /subscriptions
```
