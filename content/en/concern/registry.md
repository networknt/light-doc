---
title: "Registry Discovery"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This module contains all the interfaces needed in registry and discovery. It also implements a direct registry, which you can hard-code services into the service.yml to simulate Consul or Zookeeper during development. Although this is for local development, many users are still using it during production when they have allocated services to the exact IP and port on virtual machines.

Currently, [Consul](/concern/consul/) and [ZooKeeper](/concern/zookeeper/) are supported for external service registry and discovery.

The following section will focus on the DirectRegistry. 

### Single URL

This is the most simple service to URL mapping—one serviceId map to exactly one service instance. 

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002
        com.networknt.apic-1.0.0: http://localhost:7003
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```
### Multiple URLs

For one serviceId, there are multiple service instances available. In this case, we need to put two or more URLs into one string separated by a comma.

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002,http://localhost:7005
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

### With Tag

To support [multi-tenancy](/design/multi-tenancy/), we sometimes need to pass in an environment tag to lookup services. In this case, we need to put the environment parameter into the registered URLs.

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.portal.command-1.0.0: https://localhost:8440?environment=0000,https://localhost:8441?environment=0001,https://localhost:8442?environment=0002
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

### Summary

With the above examples, users can use the DirectRegistry for development or production. Here is an example of several services. 

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002,http://localhost:7005
        com.networknt.apic-1.0.0: http://localhost:7003
        com.networknt.portal.command-1.0.0: https://localhost:8440?environment=0000,https://localhost:8441?environment=0001,https://localhost:8442?environment=0002
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```


