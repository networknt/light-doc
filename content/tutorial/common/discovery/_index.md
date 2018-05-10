---
title: "Registry and Discovery"
date: 2017-11-06T17:45:44-05:00
categories: []
keywords: [tutorial]
aliases: []
toc: false
draft: false
---


Services typically need to call one another. In monolithic applications, services invoke one another 
through language-level method or procedure calls. In a traditional distributed system deployment, services 
run at fixed, well known locations (hosts and ports) and so can easily call one another using HTTP/REST or 
some RPC mechanism. However, a modern microservice-based application typically runs in a virtualized or 
containerized environments where the number of instances of a service and their locations changes dynamically.

This tutorial shows how to use the service registry and discovery feature of light platform.

* [Introduction and code generation](generated)

* [Static Configuration](static)

* [Dynamic service discovery with direct registry](dynamic)

* [Multiple Instances of the same service](multiple)

* [Consul service registry and discovery](consul)

* [Multiple instances with tags per environment](tag)

* [Access consul with acl_token for security](token)

* [Service discovery for Docker container](docker)

* [Service discovery for Kubernetes](kubernetes)

