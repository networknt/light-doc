---
title: "Light OAuth 2.0 Services"
date: 2019-04-30T16:06:17-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [started the consul][] server locally, now let's start the light-oauth2 services as they are part of the portal. We will start more services later on, but here we use the light-oauth2 services to demo the process. 

We cannot use the docker-compose-oauth2-mysql.yml in light-docker as the configuration there is not using the consul registry. In order to use the consul registry and discovery, all services should be bound to the host network instead of the docker network.  

To use the consul registry, I have created another folder in the light-config-test repository. The docker-compose file and configuration files are copied from the light-docker. 


```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose up -d
```

If this is the first time you start the docker-compose, it will take a while to download all images. Once all service is up, go to the consul UI again to check if all services are registered. 

```
http://lightapi.net:8500/ui/dc1/services?status=passing
```

You should find seven services registered on the consul, and they are all healthy. 

The light-oauth2/local-consul config files are copied from the light-docker/light-oauth2/mysql with the following modifications. 

### client.yml

Add client.yml, client.keystore, and client.truststore to the config folder so that the client module can be used to connect to the local consul server. 

There is no need to add these files as we are not using TLS to connect to consul; however, it is always a good idea to add these files just in case we move to the official test environment with a Consul cluster only offer HTTP/S connection. 

### secret.yml

If you are using light-4j version 1.6.x with Java 8, then we need to add the consulToken that matches the master token used in the consul container. 

```
consulToken: the_one_ring
```

If you are using light-4j version 2.0.x and above with Java 11, then you need to add this line to the consul.yml file below. 

### server.yml

In this config file, we need to change the enableRegistry to true. Since all light-oauth2 services have their port numbers allocated, we don't need `dynamicPort` to be enabled. 

```
enableRegistry: true
```

### service.yml

We need to add the section for the registry, load balance, and cluster support at the beginning of the file. 

```
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: consul
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

For all services, it needs to connect to the MySQL database. Let's update the jdbcUrl with lightapi.net as the hostname. 

```
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      jdbcUrl: jdbc:mysql://lightapi.net:3306/oauth2?useSSL=false&disableMariaDbDriver
      username: mysqluser
      password: mysqlpw
      maximumPoolSize: 2
      useServerPrepStmts: true
      cachePrepStmts: true
      cacheCallableStmts: true
      prepStmtCacheSize: 4096
      prepStmtCacheSqlLimit: 2048

```

### consul.yml

We also need to create a new file to specify the parameters for ConsulClient.

```
# Consul URL for accessing APIs
consulUrl: http://lightapi.net:8500
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: true
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
```

Here you can see that we are using lightapi.net as the hostname in the `consulUrl` and `httpCheck` is set to true. 

### docker-compose.yml

We need to update the copied docker-compose.yml to switch to the host network. 

Here is one of the services. 


```
    oauth2-token:
      image: networknt/oauth2-token
      ports:
        - "6882:6882"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-token:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
```

The network mode is set as `host` and a new environment variable is added to map the `STATUS_HOST_IP` to `lightapi.net`.

For the source code of the docker-compose and config files, please visit https://github.com/networknt/light-config-test/tree/master/light-oauth2/local-consul

In the next step, we are going to [start the router][] instance to serve the SPA and proxy the request from the SPA to light-oauth2 services. 

[started the consul]: /tutorial/portal/local-router/start-consul/
[start the router]: /tutorial/portal/local-router/light-router/
