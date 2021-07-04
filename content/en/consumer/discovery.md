---
title: "Service Discovery"
date: 2018-03-01T21:40:21-05:00
description: ""
categories: []
keywords: []
weight: 180
aliases: []
toc: false
draft: false
reviewed: true
---

When deploying microservices to a Kubernetes cluster in the cloud, there is no static IP and port number for these services anymore. In order to invoke these services, you either use the proxies provided by your Kubernetes cluster vendor or handling it with the Light service registry and discovery on the host network. 

We have done a performance test with one of our customers on Openshift cluster and found the following interesting numbers with one of the services built on top of light-rest-4j. 

### Use Openshift Proxies

* Throughput: 2450
* Latency: 68ms

### Use Consul Service Discovery

* Throughput: 20000
* Latency: 6ms

As you can see, when proxies are used in Openshift, the entire cluster can only achieve a total throughput of about 2500 requests per second. Imagine that you have hundreds of services share the same throughput, then each service will only have several requests per second.

The above reason is why we have to provide our own service registry and discovery so that our services can talk to each other directory without going through any proxy. That means we don't need any extra network hop. Given we have direct connection with HTTP 2.0 which support multiplexing, we can cache the connection for a number of minutes or a number of requests to further speed up the interaction of services. 

### Summary

There are several open source products that support service registry and discovery on the market. We have implemented module in light-4j to support both [Consul][] and [Zookeeper][].

[Consul]: /consumer/consul-discovery/
[Zookeeper]: /consumer/zookeeper-discovery/
