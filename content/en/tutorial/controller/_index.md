---
title: "Light Controller Tutorial"
date: 2020-12-04T20:12:21-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The light-controller is part of the light-portal, and it is responsible for managing live instances in a microservices deployment in real-time. For most users who cannot deploy the light-portal or just want to set up an environment for development, we have peeled the light-portal functionality and created a free open source edition. The published docker image is free to use with a docker-compose.yml in the light-controller repository for users who prefer Docker.


* [Petstore Registry][] to show users how to the light-controller for service interaction.

* [Service Discovery][] to show users how to leverage the light-controller for global registry and discovery. 

* [Kubernetes Cluster][] to show users how to use the light-controller in a Kubernetes cluster for instance management and goblal discovery. 

* [Local IDE][] to integrate light-scheduler, light-controller and petstore API

* [Local Docker][] to integrate light-scheduler, light-controller and petsotre API with multiple nodes docker-compose.


[Petstore Registry]: /tutorial/controller/petstore-registry/
[Service Discovery]: /tutorial/controller/service-discovery/
[Kubernetes Cluster]: /tutorial/controller/kubenetes-cluster/
[Local IDE]: /tutorial/controller/local-ide/
[Local Docker]: /tutorial/controller/local-docker/
