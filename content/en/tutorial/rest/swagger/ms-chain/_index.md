---
title: "Microservices Chain Pattern"
date: 2017-11-29T10:05:35-05:00
description: "How to build an application with a chain of restful microservices"
categories: []
keywords: [tutorial]
toc: false
draft: false
---

These days light weight container like Docker is getting traction, more and more 
API services are developed for docker container and deployed to the cloud. In this
environment, traditional heavy weight containers like Java EE and Spring are 
losing ground as it doesn't make sense to have a heavy weight container wrapped 
with a light weight docker container. Docker and container orchestration tools 
like Kubernetes and Docker Swarm are replacing all the functionalities Java EE 
provides without hogging resources.

There is an [article](https://www.gartner.com/doc/reprints?id=1-3N8E378&ct=161205&st=sb) 
published by Gartner indicates that both Java EE and .NET are declining and will
be replaced very soon. 


Another clear trend is standalone Gateway is phasing out in the cloud environment 
with docker containers as most of the traditional gateway features are replaced 
by container orchestration tool and docker container management tools. In addition, 
some of the cross cutting concerns gateway provided are addressed in API frameworks
like light-4j and related light*4j frameworks.


This tutorial shows you how to build 4 services chained together one by one. And it will
be the foundation for our microservices benchmarks.

API A -> API B -> API C -> API D

It is a very long tutorial as we want to package as much info as possible. So we have
separated it into the following steps. 


#### If you would like to gain a general understanding of how this application will work visit our **[Quick Start][]** to read a step-by-step of how our pre-built Microservices Chain example works.


1. [Environment preparation][]

2. [Create specification][]

3. [Generate and Build][]

4. [Http Chain][]

5. [Performance with Http][]

6. [Https Chain][]

7. [Performance with Https][]

8. [Security][]

9. [Metrics][]

10. [Docker compose][]

11. [Docker compose performance][]

12. [Kubernetes][]


[Environment preparation]: /tutorial/rest/swagger/ms-chain/preparation/
[Create specification]: /tutorial/rest/swagger/ms-chain/specification/
[Generate and Build]: /tutorial/rest/swagger/ms-chain/generation/
[Http Chain]: /tutorial/rest/swagger/ms-chain/httpchain/
[Https Chain]: /tutorial/rest/swagger/ms-chain/httpschain/
[Security]: /tutorial/rest/swagger/ms-chain/security/
[Metrics]: /tutorial/rest/swagger/ms-chain/metrics/
[Performance with Http]:  /tutorial/rest/swagger/ms-chain/httpperf/
[Performance with Https]: /tutorial/rest/swagger/ms-chain/httpsperf/
[Docker compose]: /tutorial/rest/swagger/ms-chain/compose/
[Docker compose performance]: /tutorial/rest/swagger/ms-chain/composeperf/
[Kubernetes]: /tutorial/rest/swagger/ms-chain/kubernetes/
[Quick Start]: /tutorial/rest/swagger/ms-chain/quickstart
