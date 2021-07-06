---
title: Consul Service Registry and Discovery
linktitle: Consul Service Registry and Discovery 
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: "Use Consul as a resilient and fault tolerant registry for service discovery by service id."
weight: 50
sections_weight: 50
draft: false
toc: true
reviewed: true
---

## Introduction

In this step, we are going to use Consul for registry to enable cluster scaling and load balancing.

First, let's copy our current state from last step into the `consul` directory.

```bash
cd ~/networknt
cp -r light-example-4j/discovery/api_a/multiple light-example-4j/discovery/api_a/consul
cp -r light-example-4j/discovery/api_b/multiple light-example-4j/discovery/api_b/consul
cp -r light-example-4j/discovery/api_c/multiple light-example-4j/discovery/api_c/consul
cp -r light-example-4j/discovery/api_d/multiple light-example-4j/discovery/api_d/consul
```

## Configuring the APIs


### API A

To switch from direct registry to consul registry, we need to update `service.yml` configuration to inject the consul implementation to the registry interface.

Let's make the following change in `service.yml`:

```yaml
singletons:
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

Although in our case, there is no caller service for `API A`, we still need to register it to consul by enabling it in `server.yml`. 

Let's make the following changes in `server.yml`: 

```yaml
enableRegistry: true
```

We also need to create a new consul.yml file that defines the parameters for Consul API and health check options. 

```yaml
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
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
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: true
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false
```

### API B

Let's update `service.yml` to inject the consul registry instead of the direct registry used in the previous step.

service.yml

```yaml
singletons:
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

As `API B` will be called by `API A`, it needs to register itself to consul registry so that `API A` can discover it through the same consul registry. To do that you only need to enable server registry in the server.yml config file and disable HTTP connection.

server.yml

```yaml
enableRegistry: true
```

We also need to create a new consul.yml file that defines the parameters for Consul API and health check options. 

```yaml
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
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
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: true
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false
```

### API C

Although `API C` is not calling any other APIs, it needs to register itself to consul so that `API A` can discovery it from the same consul registry.

`service.yml`

```yaml
singletons:
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

`server.yml`

```yaml
enableRegistry: true
```

We also need to create a new consul.yml file that defines the parameters for Consul API and health check options. 

```yaml
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
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
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: true
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false
```


### API D

`service.yml`

```yaml
singletons:
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

`server.yml`

```yaml
enableRegistry: true
```

`consul.yml`

```yaml
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
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
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: true
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false
```

## Start Consul

Here we are starting consul server in docker to serve as a centralized registry. To make it simpler, the ACL in consul is disabled by setting acl_default_policy=allow.

```bash
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

## Starting the servers

Now let's start four terminals to start servers.  

**API A**

```bash
cd ~/networknt/light-example-4j/discovery/api_a/consul
mvn clean install -Prelease
java -jar target/aa-1.0.0.jar
```

**API B**

```bash
cd ~/networknt/light-example-4j/discovery/api_b/consul
mvn clean install -Prelease
java -jar target/ab-1.0.0.jar
```

**API C**

```bash
cd ~/networknt/light-example-4j/discovery/api_c/consul
mvn clean install -Prelease
java -jar target/ac-1.0.0.jar
```

**API D**

And start the first instance that listen to 7445 as default

```bash
cd ~/networknt/light-example-4j/discovery/api_d/consul
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

Now you can see the registered service from Consul UI.

```bash
http://localhost:8500/ui
```

![discovery-consul-ui](/images/discovery-consul-ui.png)


## Testing the servers

```bash
curl -k https://localhost:7441/v1/data
```

And the result will be

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```
 
## Adding another API D
 
Now let's start the second instance of API D. Before starting the server, let's update `server.yml` with port 7444.

```yaml
httpsPort:  7445
```

And start the second instance.

```bash
cd ~/networknt/light-example-4j/discovery/api_d/consul
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

After you start the second instance of API D, you can go to the Web UI of consul to check if there are two instances for the API D. 

### Test Servers

Let's issue the same command again. 

```bash
curl -k https://localhost:7441/v1/data
```

And the should be either of the following:

```bash
["API D: Message 1 from port 7445","API D: Message 2 from port 7445","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]

["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Shut down the server with port 7445. Now when you call the same curl command, you will see 7444 server picks up the request.  

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

In this step we have used Consul for API registration. When we call `API A `, all other APIs were discovered from Consul to fulfill the request. In the next step, we are going to start [multiple][] instances that represent different environments differentiated by [tags][].

[multiple]: /tutorial/common/discovery/multiple/
[tags]: /tutorial/common/discovery/tag/
