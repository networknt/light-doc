---
title: "Proxy Client"
date: 2018-05-29T18:39:45-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


This is a simple proxy client for light-router, and it can discover and load balance the hosts from Consul or other service discovery services supported by the light platform.

As the target servers are implemented in light-4j which is HTTP 2.0 supported, there is no need to have a connection pool to be implemented. For each host, there would be only one connection as HTTP 2.0 supports multiplexing.

The RouterProxyClient implements Undertow ProxyClient interface, and it provides client-side service discovery, cluster support and load balance which are very specific to support microservices deployed in the cloud with Kubernetes.  It depends on two headers passed from the client to decide which connection is used to route the request. 

* service_id: The target serviceId which is unique for the service defined in server.yml for the request
* env_tag: The environment tag is used for [environment segregation][] 

An HTTP2 connection is created on the first request to the target server and cached in a hash map for subsequent request to look up. 

[environment segregation]:  /design/env-segregation/



