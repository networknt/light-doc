---
title: "GraphQL Tutorial"
date: 2018-01-03T09:18:56-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 50
weight: 50
aliases: []
toc: false
draft: false
---

GraphQL is gain traction these day as a RESTful replacement for Mobile native applications
and Single Page Applications built on top of React, Vue and Angular. It is designed to resolve
common issues with RESTful API and has a lot of wonderful features; however, for most developers
it is very hard to learn in the first place. In this section, we are going to walk through several
light-graphql-4j examples and show you how to build your own GraphQL server. 

There are three tutorials and source code can be found at light-example-4j [graphql folder][].
 
### [Hello World][]

This is a very simple Hello World query to show you how to get GraphQL up and running with
light-codegen without using GraphQL IDL.

### [Star Wars][]

This is a similar example as Hello World with IDL to trigger the generation. It is utilize the
star wars GraphQL IDL downloaded from the Internet.
 
### [Mutation][]

This example shows you how to create a full blown GraphQL service with both query and mutation.

### [Relay Todo][]

This is to show you how to build a GraphQL service that is working with Relayjs. 


[graphql folder]: https://github.com/networknt/light-example-4j/tree/master/graphql
[Hello World]: /tutorial/graphql/helloworld/
[Star Wars]: /tutorial/graphql/starwars/
[Mutation]: /tutorial/graphql/mutation/
[Relay Todo]: /tutorial/graphql/relay-todo/
