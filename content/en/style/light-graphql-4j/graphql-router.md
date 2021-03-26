---
title: "Graphql Router"
date: 2017-11-29T21:00:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The Graphql Router is responsible for handling GraphQL and GraphiQL requests while hooking schema providers. It provides RouteHandler and SchemaProvider interfaces and implements both GET and POST handlers for GraphQL.

The router is a HandlerProvider that needs to be put into the service.yml config file.

This [link][] is an example.

The user developed schema needs to be hooked to the GraphqlPostHandler in this module through SchemaProvider interface. The service.yml config file should include the implementation of the com.networknt.graphql.router.SchemaProvider


[link]: https://github.com/networknt/light-example-4j/blob/master/graphql/mutation/src/main/resources/config/service.yml
