---
title: "Light Router Tutorial"
date: 2018-02-11T16:30:29-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

API consumers can use Light-router if they don't support Java 8/11, or they are implemented with other programming languages in the service to service communication. It is the service mesh feature like other service mesh providers; however, we only need a Light router for consumers built on other frameworks other than light-4j, which has an embedded gateway in every instance. Best of all, the gateway in light-router or light-4j instance can be easily extended or customized as they are all designed as pluggable [middleware handlers](/architecture/middleware-handler/) to address certain [cross-cutting concerns](/concern/). With more flexibility and extendability than other service-mesh solutions, we call it [service-mesh-plus](/architecture/service-mesh-plus/)

The other scenario is to deploy light-router to the DMZ to act as an API gateway for traffic from the Internet. The light-router can handle all the gateway functions as well as supporting Desktop, SPA and Mobile applications. 


There are two different locations that the router can be deployed. For more details, please refer to [light-router location and ownership][].

For vary options of light-router, please refer to [light-router configuration][] 

The tutorials are still working in progress, and more will be added later.


- [Deploy light-router to cloud server](/tutorial/common/discovery/router/)
- [Deploy light-router for taiji-faucet service](/tutorial/router/taiji-faucet/)
- [Add a react single page application to taiji-faucet router](/tutorial/router/router-spa/)
- [Virtual hosts on taiji-faucet router](/tutorial/router/virtual-host/)
- [Virtual hosts on light-portal](/tutorial/router/light-portal/)
- [Live Examples of light-router](/tutorial/router/live-example/)

[light-router location and ownership]: /service/router/location-ownership/
[light-router configuration]: /service/router/configuration/

