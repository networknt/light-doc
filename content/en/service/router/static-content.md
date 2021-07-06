---
title: "Static Content"
date: 2018-11-03T10:05:33-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In most cases, light-4j microservices will be deployed to a Kubernetes cluster in the cloud with dynamic IPs and ports. Web applications consuming these services must be able to discover the services based on the serviceIds. The light-router is built for the exact purpose. 

It provides a static IP access point to all the microservices in the cloud with service discovery and security addressed so that a single page application running on the browser can access the backend services through APIs. 

Most single page applications will consume different endpoints from services serving by different domains. If the access is direct, this requires all microservices to have CORS enabled and sometimes it is hard to do as these services might be owned by different companies. 

The light-router in this scenario can be deployed as a microservices gateway to router traffic from SPA to other services. From the SPA point of view, all endpoints are served by the same domain of light-router. This pattern is different than the traditional API gateway as the microservices gateway serves to one and only one client and it can be scaled infinitely based one the SPA traffic. 

Another essential feature for light-router is to serve the static content for the single page application. Since the same domain serves the SPA and the API, there is no need to enable CORS on the light-router, and this is a significant performance boost for your SPA. 

For each access to an API not on the same domain as the SPA, there are two requests sent to the server, and the first one is the preflight request. If you have a slow network, this could be a considerable burden for the SPA application. Imagine your SPA application making millions of requests for different domains. It makes perfect sense to serve the SPA from the same domain to avoid CORS. 


It is straightforward to set up [light-router to serve SPA][] as all necessary components are built-in, and the only thing you need to do is the configuration. Besides, you can even set up your light-router instance to [support virtual host][] so that you can serve multiple SPAs from the same router instance. 



[light-router to serve SPA]: /tutorial/router/router-spa/
[support virtual host]: /tutorial/router/virtual-host/
