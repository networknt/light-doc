---
title: "Design First vs Code First"
date: 2018-02-13T20:50:25-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "design"
    weight: 5
weight: 5
aliases: []
toc: false
draft: false
reviewed: true
---

When it comes to using REST API specification formats, two important approuches have emerged: 
The “Design First” and the “Code First” approach to REST API development.

The Code First approach is a more traditional approach to building APIs, with the development of code happening after the business requirements are laid out, eventually generating the documentation from the code. The Design First approach advocates for designing the API’s contract first before writing any code. This is a relatively new approach, but is fast catching on, especially with the use of OpenAPI specification formats.

* Design First: The requirement is converted to a human and machine readable contract, such as a 
Swagger document, from which the code is built or generated.

* Code First: Based on the business requirement, API is directly coded, from which a human or machine 
readable document. The Swagger document can be generated later from the code.  

In the following section, we are going to discuss the pros and cons of each approach. We are also going to discuss which approach should be used given your situation.  

## Design First Approach

#### When Developer Experience Matters

A well designed API can do wonders for the adoption and consumption of your APIs, and good design 
can be better achieved with the Design First approach. If your API strategy involves high adoption 
of your API and retention of users integrating with your API, then good Developers Experience (DX) 
matters. An effective API design helps your end consumers quickly understand your API’s resources 
and value propositions, reducing the time taken for them to integrate with your API. An API with 
consistent design decreases the learning curve when integrating with your API, making it more likely 
to have higher reuse value and engagement.


#### When Delivering Mission Critical APIs

The biggest reason to go for the Design First approach is when your API’s target audience is external customers or partners. In such a case, your API is a key distribution channel that your end customers can use to consume the services you provide, and good design plays a key role in determining customer satisfaction. Such APIs play a critical role in representing your organization’s services, especially in an omni-channel ecosystem, where consistency in information and hassle-free consumption is an important indicator of business success.

#### When Ensuring Good Communication

The API contract can act as the central draft that keeps all your team members aligned on what your 
API’s objectives are, and how your API’s resources are exposed. Identifying bugs and issues in the 
API’s architecture with your team becomes easier from inspecting a human-readable design. Spotting 
issues in the design, before writing any code is a much more efficient and streamlined approach, 
than doing so after the implementation is already in place.    

#### When Break A Monolithic to Micorservices

When you are breaking a big monolithic application into microservices, you need to design the contracts between each service so that separate teams can work on these services independently in order to increase productivity. The contract will help teams in communication and ensure that the developed services can be integrated together.

#### When Using Non-REST API

When people talk about design first or code first, they are talking about REST APIs as most REST API frameworks provide some sort of document generation from the code. If you are building your API with GraphQL or RPC, then there is no choice. You have to write the schema or proto files and a good framework will help you in scaffolding the project from the contract.
 
#### When You Want To User the API Spec as Interface Agreement Document

Most enterprises will adopt API specification as an Interface Agreement document or make it part of the IA so that you don’t need to describe the interface in Word or Excel documents. The traditional way that developers read Word or Excel documents and convert requirements into code is long gone as there are a lot of misunderstandings in natural language to describe the API. Specifications are designed to clearly describe the contract for both human and computer and it is a live document which is always in sync with code if design first is adopted.

#### When The Chosen Framework Provide A Code Generator

It makes perfect sense to create the specification first and then use a code generator to scaffold the project. It saves a lot of time to start a project from scratch and copying/modifying existing projects might introduce some bugs that are very hard to detect. Most of the generators will offer features to re- generate the code with the same specification as the framework has been upgraded to a new version. Even if the specification has been changed, you can still generate a new project and then do a full text comparison with the old codebase to merge your code into the new project.

#### When The Framework Can Leverage The Spec During Runtime

Some of the frameworks like [light-rest-4j][], [light-graphql-4j][] and [light-hybrid-4j][] can load the specification during runtime and use it for request validation and scope verification based on JWT tokens. In this case, the contract is not just for code generation but is part of the application to enforce the contract during the runtime.

#### When You Want Consumer To Work In Parallel

Another benefit of the design first approach is that you can have the specification to generate the code with examples defined for each endpoint. After the code generation, you will have a running mock API that you can dockerize and deliver to the consumer team to be worked with. This can facilitate the consumer and provider to work in parallel with maximum productivity. During the interaction, the consumer teams might give us feedback quickly to iterate the API design and then redeliver the new image to the consumer.

#### When You Have Marketplace To Publish APIs

If you have a marketplace deployed in your organization, the API design can be published to it as soon as the first draft is released. This will attract potential consumers and other teams to be involved with API design in a very early stage. The feedback from these reviewers will greatly help with the API design team in stabilizing the contract.

#### When You Adopt Consumer Driven Design

Some organizations adopt the consumer driven contract design which means all potential consumers will be engaged and asked to provide requirements about the API interface; the provider interface will be an aggregation of all potential consumers requirements. Sometime, as the provider has more knowledge about the data and business domain, they might provide more features than all consumers asked for. The design specification is the crucial piece in the communication between teams in this case.

#### When Your Test Team Is Working In Parallel

In order to deliver microservices to production as fast as possible, we need to have the CI/CD pipeline be built in the DevOps tool chain and unit tests, integration tests, consumer contract tests and end-to-end tests. This is very important to ensure the quality of the service and give management confidence to continuous integration to production. In order to engage the QA team as early as possible, a detailed design specification is a must. If you want to engage the QA team after you complete your coding, it will be too late. 

## Code First Approach

#### When Delivery Small Independent REST API

When you are working on a single, simple REST API, sometimes, you can skip the design and start coding immediately. Once the code is done, you can generate the OpenAPI specification from the code or write it just like a documentation for consumers. In most of the cases, you are going to pick up an existing example project and then make some modifications on top of it. For big APIs, we might not see some of the design flaws early enough and they may appear only after one or two versions are released to production.

#### When The API Framework Doesn't Support Code Generation

When the chosen API framework doesn’t provide a code generator, it makes sense to write the code first. Chances are, there is a document generator to generate the specification from comments and annotations in the code. Even though you have produced the design document, developers still need to read the design and translate it into the code.

## Summary

There are positives and negatives to both approaches. However, for enterprise scale API development design first is recommended. As Light is aimed at enterprise microservices architecture, we encourage users to use a design first approach as we have [light-codegen][] to generate the project. We can also use the light-codegen to regenerate the same project to upgrade the framework version. In addition, we just talked about REST API in this article as the people who asked questions are focusing on REST API only. If we use the GraphQL or RPC style of API, you have no choice but to create the schema first and generate code with light-codegen. There is no code first for other styles of APIs. All in all, code first is for an individual developer who wants to create an ad hoc API out of his/her mind and get it up and running first and then rethink about design later.

 
[light-rest-4j]: /style/light-rest-4j/
[light-graphql-4j]: /style/light-graphql-4j/
[light-hybrid-4j]: /style/light-hybrid-4j/
[light-codegen]: /tool/light-codegen/
[consumer driven contract]: /design/consumer-contract/
