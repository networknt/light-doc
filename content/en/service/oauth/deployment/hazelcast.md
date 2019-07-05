---
title: "Hazelcast Security"
date: 2019-07-05T10:54:43-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-oauth2 consists of 8 microservices and these services communicate between each other through a Hazelcast distributed cache. All services are working at the cache layer so that it is very easy to abstract the persistence layer to any database. Currently, we only support MySQL, Postgres, Oracle, and SQLServer. And these can be exchanged by configuration. User can extend the persistence layer to support other SQL or No-SQL database. 

By working at the Data Grid level, we can hide the persistence from the services. At the same time, it significantly improves the performance of light-oauth2 services. However, there are some extra configurations we have to consider to secure the Hazelcast. 

When talking about Hazelcast security, there are two different areas: 

Authentication/Authorization - which application can connect to the Hazelcast IMDG and client permission. 

For more information, please visit https://github.com/hazelcast/hazelcast/issues/8370

Multicast Security - Ensure nobody can intercept the network traffic. It requires TLS to encrypt the connection. 

Hazelcast enterprise has features to meet the security requirement above; however, most of our users won't want to buy the enterprise license. So we have to find a way to secure the Hazelcast at the network level. 


The light-oauth2 is an infrastructure service, and it is normally not recommended deploying to the Kubernetes cluster; however, more and more users are doing so. So let's separate the deployment from VM and Kubernetes/Openshift. 

### VM deployment

When deploying the light-oauth2 to one or several VMs, you can start the containers with docker-compose or docker-swarm. These VMs are secured by firewalls so that Hazelcast communication can only be allowed between these VMs. Only the service ports should be exposed to the outside through either static IP with port numbers or through Consul service registry and discovery. 


### Kubernetes/Openshift

When deploying the light-oauth2 to a Kubernetes/Openshift cluster, it is recommended deploying light-oauth2 services to a separate cluster so that these nodes can be secured, or at least deploying to a separate namespace/project so that security can be configured separately. 

The network policy configuration in Kubernetes can be found at https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource, and we will write some tutorial in the future. 







