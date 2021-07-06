---
title: "Jar Files"
date: 2018-05-07T12:30:59-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When using the client module from any Java application built on top of Java 8 but not on Light, several modules need to be included together to perform multiple functions including HTTP/2 request, service discovery, load balance and cluster support. These jar files are tiny which only contain 2 to 5 classes. The following list includes all the related jar files from Maven Central when using client module. 

For all the modules listed below, there are three configuration files that need to be externalized in order to work. There are: 

1. client.yml - defines client configuration and OAuth 2.0 communication. It is used by client module only.
2. service.yml - balance, cluster, registry, consul and decryptor configurations are defined in this file. 
3. secret.yml - OAuth 2.0 client secret and certificate password etc. are defined in this file for easy secret management. All secrets can be encrypted so that no clear text can be seen in the config file. 




* [client.jar][]

It is the main jar file that is responsible for creating an HTTP/2 connection, sending requests and invoking client callbacks. It also handles auto-renewal of JWT tokens from the OAuth 2.0 provider and passes through the correlationId and traceabilityId.

* [config.jar][]

It loads configuration files inside the module, application or externalized directory.

* [balance.jar][]

It is the module to handle the client side load balance. It supports three built-in strategies: Round Robin, Local First, and Consistent Hashing.

* [cluster.jar][]

Cluster support at the client side, it uses balance module to pick up one candidate (IP and port) from a list of discovered instances.

* [registry.jar][]

It Provides general service discovery logic along with concrete implementations like Consul or Zookeeper.

* [consul.jar][]

It is a client module for Consul agent communication. 

* [service.jar][]

Dependency injection service that is responsible for wire-in all other modules.

* [decryptor.jar][]

It is an optional module if you want to encrypt sensitive data from the configuration files.

* [common.jar][]

It is an optional module and used by decryptor for the DecryptUtil.

[client.jar]: /concern/client/
[config.jar]: /concern/config/
[balance.jar]: /concern/balance/
[cluster.jar]: /concern/cluster/
[registry.jar]: /concern/registry/
[consul.jar]: /concern/consul/
[service.jar]: /concern/service/
[decryptor.jar]: /concern/decryptor/
[common.jar]: /concern/common/

