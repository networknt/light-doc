---
title: "Spring Boot Integration"
date: 2019-03-06T17:00:50-05:00
description: ""
categories: [style]
keywords: []
menu:
  docs:
    parent: "style"
    weight: 06
weight: 06
aliases: []
toc: false
draft: false
---

Recently, one of the big customers asked if we can integrate light-4j middleware handlers to address cross-cutting concerns to Spring Boot applications. They want to leverage the light-4j middleware handlers as they are faster, smaller and easier to implement and customize. Most importantly, these handlers working with light platform infrastructure to form an ecosystem to support microservices architecture. In this way, they can migrate their existing Spring Boot applications to the light platform without changing the controllers and business logic. The different development teams welcome the idea as they can still use Spring Boot to develop or maintain their applications without knowing the light-4j middleware injections as the DevOps team handles it in the pipeline. 

The latest Spring Boot 2.1.x can have embedded Undertow Servlet Web Server to serve the HTTP request, and it provides a way to customize the DeploymentInfo which can be used to inject the light-4j middleware handlers before passing the control back to ServletInitialHandler. The ServletInitialHandler is the entry point to all the Spring Boot controllers which handles the mappings for the restful endpoints. 

Along with embedded servlet, Spring Boot 2.0 also provides an embedded reactive web server to support async IO with better throughput. It is called WebFlux, and the programming model is basically reactive. In the long term, we need to support this type of application to inject light-4j handlers as well. However, there is no demand at the moment, so we only implement the solution for the servlet in the first release. 

* [Getting-Started](/tutorial/springboot/)
* [Performance](/benchmark/spring-boot/)
* [Servlet](/style/light-spring-boot/servlet/)
* [Reactive](/style/light-spring-boot/reactive/)
* [Tutorial](/tutorial/springboot/)

