---
title: "Light Graphql 4j"
date: 2017-11-05T11:46:23-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 20
weight: 20
sections_weight: 20
aliases: []
toc: false
draft: false
---

[GraphQL][] is a new style of protocol open sourced by Facebook and it is getting popular
especially in Mobile and Single Page Application as front end. GraphQL provides a complete 
and understandable description of the data in your API, gives clients the power to ask for 
exactly what they need and nothing more, makes it easier to evolve APIs over time, and 
enables powerful developer tools.

[light-graphql-4j][] is a GraphQL framework built on top of [light-4j] and [graphql-java].

There are two options to start a graphql project with light-graphql-4j:

* Define your GraphQL IDL and generate the code from it with [light-codegen][]

* Start a new project by copying one of the [examples][] and replace the schema with yours.

For more information on these examples, please refer to [graphql tutorial][].

The most important tool for GraphQL application is GraphiQL which is an web interface that
allow you to introspect the GraphQL server and execute queries and mutations. Once you have
your light-graphql-4j application up and running, the GraphiQL is served at

http://localhost:8080/graphql

or 

https://localhost:8080/graphql

For more information about this tool, please check this [site][].

[GraphQL]: http://graphql.org/
[light-graphql-4j]: /style/light-graphql-4j/
[light-4j]:/concern/
[graphql-java]: https://github.com/graphql-java/graphql-java
[light-codegen]: /tool/light-codegen/
[examples]: https://github.com/networknt/light-example-4j/tree/master/graphql
[graphql tutorial]: /tutorial/graphql/
[site]: https://github.com/graphql/graphiql

