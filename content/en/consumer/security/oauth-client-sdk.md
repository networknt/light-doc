---
title: "OAuth Client SDK"
date: 2019-04-25T15:20:36-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

These days, most API endpoints are protected by OAuth 2.0 standard either through an API Gateway or distributed verification based on JWT tokens by the services themselves. 

In a microservices architecture, there are lots of service to service communications. The traditional Monolithic API Gateway won't work in this case as it becomes the single point of failure and bottleneck. 

In most microservices platform, JWT is used instead of simple web token which has to go to the OAuth 2.0 provider for introspection. 

We have received a lot of questions from our users regarding which is the best OAuth 2.0 client SDK in Java, Node.js or .NET. Here we are going to answer the questions based on our best knowledge. 

### Criteia

* Comply with OAuth 2.0 standard 
* Clearly document the security consequence for each grant type
* Make unsafe operations harder to use and document them.
* Wrap the interaction with OAuth 2.0 provider and provide a simple API for end users.
* Built-in HTTP/S and HTTP/2 support
* Support both JWT token and SWT token with the cache.
* Correlation and Traceability support
* Open Source with a commercially friendly license
* Has a big user base and an active developer's community
* Easy to customize


### Three categories

When searching for the OAuth 2.0 client SDK, we are mainly looking at the following three categories. 

* Standalone OAuth Client SDK. 

These are specialized for OAuth 2.0 client, and the implementation will work with standard OAuth 2.0 specification. When using it within our enterprise computing environment, some customizations are needed as every OAuth 2.0 implementation will have some extra features that we want to leverage. If we're going to leverage a standalone library, we need to make sure that the community is active and the library is easy to customize. 

* Client SDK provided by OAuth 2.0 provider

Most OAuth 2.0 providers will have a client module or library to allow users to connect to the server that implements the OAuth 2.0 specification. As there are so many specifications related to the OAuth 2.0 and different OAuth 2.0 providers will have different feature sets, the client module might not be generic and might not work out of the box for another OAuth 2.0 server implementation. We need to make sure the library we selected is easy to understand and easy to customize. 

* Client SDK provided by the API framework

For most API frameworks, they have a client module that is responsible for the client to service or service to service communication. If the framework supports OAuth 2.0 to secure the service endpoint, the client module should have built-in support for the OAuth 2.0 specification. Most of these client implementations will support multiple OAuth 2.0 providers; however, the implementation might be tightly coupled with the underline framework, and a lot of extra features might not be interested for end users who are not using the framework. When using these client modules, we need to make sure that it is customizable for our needs. 


### Java

Most Java-based OAuth 2.0 client with hundreds of stars are not updated or not released for the last two or more years. For example

googleapis/google-oauth-java-client
codepath/android-rest-client-template
wuman/android-oauth-client
dynamind/spring-boot-security-oauth2-minimal

Although updating from the author, mttkay/signpost only supports OAuth 1.0 with 556 stars. 


The most promising standalone library for Andriod is https://github.com/openid/AppAuth-Android with 831 stars, and it was last updated 10 months ago. It is highly recommended for Andriod applications to communicate with backend services projected with OAuth 2.0. 

Other client libraries like are mostly integrated with the OAuth 2.0 providers, and they are tightly coupled with the server side implementation like the oxAuth at https://github.com/GluuFederation/oxAuth.  It is an OAuth 2.0 provider with a built-in client. Since OAuth 2.0 specification is a standard, we can be sure that the client can work with other OAuth 2.0 providers. Please be aware that most OAuth 2.0 providers will have some extra features and extensions and you might need to customize the client library to leverage these features. The project has 176 stars, 20 releases, and recent updates. 

Another type of OAuth 2.0 client is part of the API platform. These clients are generic and configurable to support multiple OAuth 2.0 providers. One of the examples is light-4j Http2Client. https://github.com/networknt/light-4j/tree/master/client. The platform has 2259 stars, 80 releases, and recent updates.

Depending on our use case, the best solution might be just pick up one of the above and customize it for our needs. 

### DotNet

https://github.com/titarenko/OAuth2

This is the only active OAuth2 standalone client implementation for .NET which was updated 6 months ago with 253 stars. There are only two releases back to 2013 with version 0.8.1 and 0.8.2 which are not production ready based on the version numbers. I am not sure how .NET libraries are packaged and used by applications so the application might use the library from the GitHub.com repository directly.

When discussing the .NET OAuth 2.0 provider implementation, there is a famous one called IdentityServer4 which can be found at https://github.com/IdentityServer/IdentityServer4. This project is very active with 4316 stars and 63 releases. You can find a lot of client samples in the samples folder at https://github.com/IdentityServer/IdentityServer4/tree/master/samples/Clients/src

There is another client that is included in an OAuth 2.0 provider implementation in .NET with all the client examples for each grant types.  https://github.com/openiddict/openiddict-core. This project has 872 stars, 8 releases and the last update was 3 months ago. There is a project for all the client samples which is very useful in learning how to use each grant types. 

The .NET is a generic framework on the Windows, and I don't find any other framework that is developed with C#. 


### Nodejs

There are some Javascript SDK for the browsers, but we are not focusing on them as most enterprise users will have a BFF for the SPA. In this section, the focus will be the service to service interaction between a Nodejs service calling another service. 

https://github.com/lelylan/simple-oauth2

This is the most popular standalone library in Javascript. The document is good with examples of different grant types. It has 894 stars on GitHub.com, 24 releases, and the last update was 11 months ago. 


https://github.com/mulesoft/js-client-oauth2

The is the open source offer from Mulesoft. I can generate a client based on the APIs described with RAML. It has 312 stars, and the last update was 5 months ago. The document is good on the readme with all examples for different grant types. 

In searching for a Nodejs OAuth 2.0 providers, we found https://github.com/t1msh/node-oauth20-provider with 568 stars but no release. The last time this repository was updated 3 years ago. Other Nodejs OAuth 2.0 implementations are really small, and there is no community support. 

There are several API frameworks in Nodejs, but almost all of them are focusing on the server and middleware implementations. I couldn't find any meaningful client SDK with OAuth 2.0 support as most of them are relying on the API Gateway so that the client is not talking to the OAuth 2.0 server directly. With more and more API Gateways implemented as distributed microservice from the monolithic application, you can find the OAuth 2.0 client implementation in distributed gateways like Apigee Microgateway. If you want to implement the client OAuth 2.0 SDK, you can find the logic in these micro-gateways and build your standalone client. 

### Summary

In a long run, we want provide clients for all popular languages to connect to light-4j services. However, we don't have the resource at the moement. If our users have client in a language other than Java 8 and above, we recommend trying above libraries and customize them if necessary. For the extended features, please take a look at the light-4j Http2Client document at https://doc.networknt.com/concern/client/ and service consumer document at https://doc.networknt.com/consumer/

