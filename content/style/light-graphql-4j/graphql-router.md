---
title: "Graphql Router"
date: 2017-11-29T21:00:16-05:00
description: "Responsible for handling GraphQL and GraphiQL requests and hooks schema provider."
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This module provides RouteHandler and SchemaProvider interfaces and implement both GET and POST handlers for GraphQL.

The router is a HandlerProvider and it needs to be put into file /src/main/resources/META-INF/services/com.networknt.server.HandlerProvider in your GraphQL API/service.

This [link][] is an example.

The user developed schema needs to be hooked to the GraphqlPostHandler in this module through SchemaProvider interface. The SPI config file should be located at /src/main/resources/META-INF/services/com.networknt.graphql.router.SchemaProvider

Another [example][].

[link]: https://github.com/networknt/light-example-4j/blob/master/graphql/mutation/src/main/resources/META-INF/services/com.networknt.server.HandlerProvider

[example]: https://github.com/networknt/light-example-4j/blob/master/graphql/mutation/src/main/resources/META-INF/services/com.networknt.graphql.router.SchemaProvider
