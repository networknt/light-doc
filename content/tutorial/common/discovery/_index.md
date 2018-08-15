---
title: "Registry and Discovery"
date: 2017-11-06T17:45:44-05:00
lastmod: 2018-05-15
menu:
  docs:
    parent: "tutorial"
    weight: 10
keywords: []
toc: false
draft: false
reviewed: true
---


Services typically need to call one another. In monolithic applications, services invoke one another through language-level method or procedure calls. In a traditionally distributed system deployment, services run at fixed, well-known locations (hosts and ports) and so can easily call one another using HTTP/REST or some RPC mechanism. However, a modern microservice-based application typically runs in a virtualized or containerized environments where the number of instances of a service and their locations (IP and port) changes dynamically.

This tutorial shows how to use the service registry and discovery feature of the light platform to discover services and invoke it with the client module.

* [Introduction and Code Generation][]

* [Static Configuration][]

* [Dynamic Service Discovery with Direct Registry][]

* [Multiple Instances of the same Service][]

* [Consul Service Registry and Discovery][]

* [Multiple Instances with Environment Segregation][]

* [Securing Consul with acl_token][]

* [Service Discovery with Docker Containers][]

* [Kubernetes][]

* [Router Assisted Service Discovery][]

* [Consul Production with TLS][]

* [Consul HTTP Health Check][]

* [External Config][]

[Introduction and Code Generation]: /tutorial/common/discovery/generated/
[Static Configuration]: /tutorial/common/discovery/static/
[Dynamic Service Discovery with Direct Registry]: /tutorial/common/discovery/dynamic/
[Multiple Instances of the same Service]: /tutorial/common/discovery/multiple/
[Consul Service Registry and Discovery]: /tutorial/common/discovery/consul/
[Multiple Instances with Environment Segregation]: /tutorial/common/discovery/tag/
[Securing Consul with acl_token]: /tutorial/common/discovery/token/
[Service Discovery with Docker Containers]: /tutorial/common/discovery/docker/
[Kubernetes]: /tutorial/common/discovery/kubernetes/
[Router Assisted Service Discovery]: /tutorial/common/discovery/router/
[Consul Production with TLS]: /tutorial/common/discovery/consul-tls/
[External Config]: /tutorial/common/discovery/external-config/
[Consul HTTP Health Check]: /tutorial/common/discovery/http-health/

