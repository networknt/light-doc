---
title: "Router Http2 to Older Version Service"
date: 2019-03-01T20:05:45-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

One customer encountered an issue and we have helped them to debug and resolve the issue. Here is the situation:

* They have a backend service that generates documents based on template and REST request. It is based on light-4j 1.5.14 which is based on Undertow 1.4.x

* They deployed a light-router instance in front of the service for a third party client to access the service along with other services. The light-router instance is built on top of light-4j 1.5.30

* The client connects to the light-router with http/1.1 and https. light-router connects to the backend with http/2 and https and the backend service is listening to http/2. 

* If the client sends a post request with a French character in the body, the router timeouts, which means the response is not received from the backend service. The debug info indicates that the backend service response is already sent. There is no problem for other requests with body without French characters. 

* The application is on UAT at the time and will be released to production in 2 weeks. The solution we have is to update the light-router config to switch from http/2 to http/1.1

* After the config change on light-router, everything works fine. We don't know exactly the root cause but guess it is due to different version of Undertow instances handle the HTTP/2 differently. We have recommended upgrading the backend service to the latest version but I will take some time. 


