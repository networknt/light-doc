---
title: "API Category"
date: 2017-11-06T11:14:26-05:00
description: "API Category - Choose the right framework for the right API type"
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
---

When an organization pursues microservices, the services/APIs can be grouped
into the following categories. For each category, certain microservice style
is more suitable than others. There are several ways to categorize APIs and
here we differentiate APIs by visibility. 

## Open Web APIs

This is also called public API. APIs in this category have public routes and 
can be accessed from anywhere. Companies want to bring their business online
to attract developers to build applications against their public APIs. In this
way, they can extend their business and reach more customers than before. 

#### Rest

When building public APIs, it seems RESTful is a natural choice as it is well known 
by almost every developer and support almost every language. It is the most mature
technology and has support from a lot of commercial infrastructure and tools.

[light-rest-4j][] is designed to build microservices in RESTful style on top of 
OpenAPI specification with most cross-cutting concerns addressed and allow developers 
to focus on the domain business logic only. 

The drawback for RESTful interaction style is its complexity in the specification and
less efficient in communication. With today's smart browser with Single Page 
Application and mobile device, REST forces the client to convert the data model
into URI and send through the HTTP connection to the server with all type information
lost. The server has to transform the URI into data model with the help of Swagger
specification. The response reverses the same process. That is the reason Swagger
specification is so complicated and very hard to write compare with other IDLs.  

#### Graphql

Since Facebook open-sourced Graphql, it is getting popular in the open source community
very fast and some companies start to adopt it to replace RESTful APIs for public
APIs. Github.com is one of the examples as they are moving to Graphql from Restful.

[light-graphql-4j][] is designed to build GraphQL API with GraphQL IDL(Interface 
Definition Language). Developers just need to wire in the domain logic into the 
generated graphql schema to make the API works. Sometimes, Graphql APIs will be 
used as an aggregate API in front of several RESTful or Hybrid APIs to serve 
different consumers like Web Server, Native Mobile and Single Page Application 
on Browser.

 

## B2B APIs

This type of APIs is designed to exchange information and collaborate with partners.
Due to the close relationship with partners, there is more flexibility in term of
choosing more efficient approach to build APIs. It is easier to train developers due
to the close relationship. 

#### Rest

Rest is very popular with this type of APIs.

#### Graphql

GraphQL is gaining traction in certain business scenarios.

#### Hybrid

Another option for partner API is RPC style of API which is much more efficient than
RESTful. 

[light-hybrid-4j][] is designed to build RPC based microservices. Also, it allows you 
to build modularized monolithic services as well. You can build a monolithic service 
in the beginning and then split one or more hot services to another instance easily. In
addition, it is a serverless framework and will be the cornerstone of lightapi.net
hosting platform.


## Internal APIs

Internal APIs will only be used within the same organization and service might be shared
between LOBs. These APIs can only be accessed from servers within our data center or private cloud. We still 
follow best practices like load balancing etc, but we don't provide any public routes to 
these APIs.

#### Rest

Rest is very popular with this type of APIs.

#### GraphQL

GraphQL is gaining traction in certain business scenarios.


#### Hybrid

Hybrid is one of the frameworks that are very suitable for product APIs. It is highly
recommended. In fact, all services built in light-portal are based on the hybrid framework.

#### Eventuate

Above three microservices frameworks are all designed for request/response type of
interaction between consumers and services. In another word, the interaction style
is synchronous and it would scale to a certain point once volume increases. In a data 
rich environment, it has its drawbacks in dealing with transactions. The distributed
transaction is not the answer in the microservices world. Ultimately, each service should 
have its own database. So no shared database is allowed. In this case, the eventual 
consistency microservices framework is needed to glue all services together to allow 
data synch between them.

[light-eventuate-4j][] is designed to build microservices based on messaging, Event 
Sourcing and CQRS on top of Kafka. The interaction style is asynchronous which is based
on domain events and will ensure data consistency eventually. Combining with [light-saga-4j][],
it can handle distributed transactions between multiple services. 

Basically, light-eventuate-4j is a [service mesh][] that is deployed in an organization
as infrastructure service to support microservices. All services still need to be built
with light-rest-4j, light-graphql-4j or light-hybrid-4j to serve consumers but underline
communication between services is done through light-eventuate-4j and transaction orchestration
is done by light-saga-4j. 


## Product APIs

Product APIs are used only around one specific product and they are not supposed to be
shared. 

#### Rest

Rest is very popular with this type of APIs but it is not very efficient.

#### GraphQL

GraphQL is gaining traction in certain business scenarios. For example, the services
close to the UI. 

#### Hybrid

Hybrid is one of the frameworks that are very suitable for product APIs. It is highly
recommended.

#### Eventuate

Like Internal APIs, Eventual consistency with messaging, event sourcing and CQRS can
be used to glue all services/APIs together. And light-saga-4j can be used to orchestrate
distributed transactions between services. 

[light-rest-4j]: /style/light-rest-4j/
[light-graphql-4j]: /style/light-graphql-4j/
[light-hybrid-4j]: /style/light-hybrid-4j/
[light-eventuate-4j]: /style/light-eventuate-4j/
[light-saga-4j]: /style/light-saga-4j/
[service mesh]: /architecture/service-mesh/
