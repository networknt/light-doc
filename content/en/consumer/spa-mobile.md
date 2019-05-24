---
title: "SPA and Mobile Discovery"
date: 2018-04-30T19:58:40-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Often, when we discuss service discovery, only web servers, standalone applications and services are mentioned as clients. We seldom discuss single page application running on the browser and mobile application running on the devices as these are not considered to perform service discovery at all. 

In most cases, these applications connect from the Internet to a static load balancer like F5 and then route to several static addressed BFF instances. So even it is capable of performing discovery from SPA or Mobile, there is no need to do so as BFF instances are accessed directly with IP addresses. Moreover, BFF is responsible for discovering microservices running in the cloud with dynamic IP addresses and port numbers. 

The reason we have this kind of architecture is to address security concerns. Our internal services running in the cloud might connecting to databases, repositories or other important backend systems. They should not be accessed from the Internet directly. They reside in the internal network and can only be accessed from BFFs which are running in DMZ. 

These days, the typical design would be an API gateway acts as a BFF which handles all the external requests and performs security filtering. It works with monolithic web services but not for microservices as the single API gateway is the bottleneck and single point of failure. 

When adopting microservices architecture, we need distributed API gateways so that workload can be balanced to different instances specifically designed for each client. These distributed gateways know the client requests much better and handle the security for the client with the OAuth 2.0 provider to shield the complex logic for client developers. Ideally, these distributed gateways should be built as microservices to ensure scalability in the long run. 


In the light platform, three components can act as a distributed gateway which is known as a BFF. 


### Existing Web Server

If your APIs were built on top of the existing web server and cannot migrate to microservices architecture overnight, you can still use your web server as a gateway for services discovery. It requires that the web server is built on Java 8 and up so that the light-4j client module can be utilized to interact with the OAuth 2.0 provider and with Consul for service discovery. If the web server cannot use the embedded client module, light-router can be used. In this design, the web server acts as an aggregator as well, so that one client request might invoke several microservices in the backend. 

### Custom Aggregator

For brand new client applications, you can build an aggregator on the light-rest-4j or light-graphql-4j framework as a microservice. It can greatly simply the client invocation logic and make the client side developers lives more comfortable. This layer also hides the details of the backend services so that our APIs of backend services will not expose to the Internet directly. 

### Light Router

With HTTP 2.0 is getting popular, the needs for aggregator is not that important anymore as a client can access multiple services simultaneously without performance penalty. 

This architecture also opens the door for true microservices which means UI and API are fully combined as one service that handles a specific business function. With client-side frameworks like React or React Native, the limitation is your imagination. 



