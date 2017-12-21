---
title: "REST - Representational state transfer"
date: 2017-11-22T21:46:27-05:00
description: "Microservices framework that support both Swagger 2.0 and OpenAPI 3.0 specifications"
categories: [style]
keywords: [rest, restful]
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
---


Light platform provides a plugin architecture and dozens of middleware handlers to address cross-cutting
concerns in general. When you decide to build an API or service, you have to choose from different 
styles and corresponding framework to get started.
 
For [API categories and frameworks][] and in which situation to choose which framework, please refer to 
the [architecture section][]. 
 
Chances are you are building a public Web API and want to follow RESTful style to make sure everyone
can understand and consume it. 

Light-rest-4j is a framework that is designed to speed up RESTful API development and deployment. On
top of light-4j, it has several middleware handlers specifically designed around Swagger 2.0 and OpenAPI 
3.0 specifications. In fact, there are totally two different frameworks to support Swagger and OpenAPI
as later is not backward compatible. 



[API categories and frameworks]: /architecture/category/
[architecture section]: /architecture/
