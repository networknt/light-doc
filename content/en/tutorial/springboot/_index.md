---
title: "Spring Boot Tutorial"
date: 2019-03-06T19:47:10-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Recently, we have introduced light-spring-boot repository with two light-4j modules to inject light-4j middleware handlers into the Spring Boot application with embedded Undertow Servlet or embedded Undertow core. The injection is happening at the http core level so these handlers will have much [better performance][]

In most cases, you have already built the Spring Boot application and you want to leverage the Light ecosystem and numeric middleware handlers. If you are starting a brand new project, it is highly recommended using light-rest-4j directly. 

* [Servlet](/tutorial/springboot/servlet/)


[better performance]: /benchmark/spring-boot/
