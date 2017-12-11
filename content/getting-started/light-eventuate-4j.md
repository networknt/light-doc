---
title: "Light Eventuate 4j"
date: 2017-11-05T11:46:59-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 40
weight: 40
sections_weight: 40
aliases: []
toc: false
draft: false
---

When building services with light platform, you can use request/response approach
like [Rest][], [GraphQL][] or [Hybrid][] to serve consumers. For service to service
communication, you have two options. One is synchronous request/response over HTTP
This requires all involved services to be responsive and highly available. 
There is another way to handler interaction between services - asynchronous event driven.
light-eventuate-4j is not only event driven framework, it supports Event Sourcing and
CQRS to ensure data consistency between services. 

If you are a developer to build services, then you don't need to clone the source code
of light-eventuate-4j. You can start all the services through a docker-compose. 

The details of the steps are documented in [light-eventuate-4j tutorial][]. 



[light-docker]: https://github.com/networknt/light-docker
[Rest]: /style/light-rest-4j/
[GraphQL]: /style/light-graphql-4j/
[Hybrid]: /style/light-hybrid-4j/
[light-eventuate-4j tutorial]: /tutorial/eventuate/getting-started/

