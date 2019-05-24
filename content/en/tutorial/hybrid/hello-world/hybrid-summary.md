---
title: "Hybrid Summary"
date: 2019-04-04T10:27:01-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this Hello World tutorial, we have created a server project and two services projects. You can add more services if you want to extend the tutorial. As all services are sharing the same server instance, the entire application can be considered as a monolithic application. However, each service is developed, tested and deployed independently as a microservice. These services deployed on the same server instance can call each other with the API just like other microservices. That is why we call the light-hybrid-4j a hybrid framework that takes advantages of both monolithic and microservices. 

If you follow the steps for service1 and service2 closely, you can see that you are creating services in a virtual testing environment and then compile it into a small jar file only several KB in size. You just drop these jars to a location that the server instance can find, and these services can start to handle your requests defined in the schema file. In this sense, the light-hybrid-4j is a serverless framework. For the developers who are building services, they don't need to care about any cross-cutting concerns. The operation team needs to build a server instance with all the dependencies and cross-cutting concerns wired in. All other services will share the same server instance and cross-cutting concerns. 

In the tutorial, we just start the server from the command line. It is very easy to build the server into a docker image and load the services through a volume mapped into the container. If you want to learn how to use Docker, please continuous other tutorials for the light-hybrid-4j. 

