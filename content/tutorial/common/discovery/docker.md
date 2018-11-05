---
title: Service Discovery with Docker Containers
linktitle: Service Discovery with Docker Containers
date: 2017-10-17T21:05:20-04:00
lastmod: 2018-05-15
description: "Using service discovery patterns in a containerized environment."
weight: 80
sections_weight: 80
draft: false
toc: true
---

## Introduction

In this step, we're going to dockerize all the APIs and then use Consul for service registry. To make it easier, we are going to use docker-compose to put everything together.

First, let's copy from consul to consuldocker for each API.
 
```bash
cp -r ~/networknt/discovery/api_a/consul ~/networknt/discovery/api_a/consuldocker
cp -r ~/networknt/discovery/api_b/consul ~/networknt/discovery/api_b/consuldocker
cp -r ~/networknt/discovery/api_c/consul ~/networknt/discovery/api_c/consuldocker
cp -r ~/networknt/discovery/api_d/consul ~/networknt/discovery/api_d/consuldocker
```

## Configuring the APIs 

### API A

We will be running in a dockerized environment, so having consul configured at "localhost" is no longer valid. We will be using the DOCKER_HOST_IP in the consul.yml file. On my local computer, the IP is 192.168.1.120. You can use ifconfig on Linux to find out your local IP. 

When starting the docker-compose for services, we need to set up the DOCKER_HOST_IP as environment variable to pass it into the container. For more info on how to setup the DOCKER_HOST_IP, please refer to [docker host ip][]. 

`consul.yml`

```yaml
# Consul URL for accessing APIs
consulUrl: http://192.168.1.120:8500
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
```

The Dockerfile generated from light-codegen should be available as following.

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
ADD /target/apia-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build and tag a docker image for this service: 

```bash
cd ~/networknt/discovery/api_a/consuldocker
mvn clean install -DskipTests
docker build -t networknt/com.networknt.apia-1.0.0 -f docker/Dockerfile .
```

### API B

`consul.yml`

```yaml
# Consul URL for accessing APIs
consulUrl: http://192.168.1.120:8500
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
```

The Dockerfile generated from light-codegen should be available as following.

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
ADD /target/apib-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build and tag the docker image for this service:

```bash
cd ~/networknt/discovery/api_b/consuldocker
mvn clean install -DskipTests
docker build -t networknt/com.networknt.apib-1.0.0 -f docker/Dockerfile .
```

### API C

`consul.yml`

```yaml
# Consul URL for accessing APIs
consulUrl: http://192.168.1.120:8500
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
```

The Dockerfile generated from light-codegen should be available as following.

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
ADD /target/apic-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build and tag the docker image for this service:

```bash
cd ~/networknt/discovery/api_c/consuldocker
mvn clean install
docker build -t networknt/com.networknt.apic-1.0.0 -f docker/Dockerfile .
```

### API D

`consul.yml`

```yaml
# Consul URL for accessing APIs
consulUrl: http://192.168.1.120:8500
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
```

The Dockerfile generated from light-codegen should be available as following.

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
ADD /target/apid-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build and tag the docker image for this service:

```bash
cd ~/networknt/discovery/api_d/consuldocker
mvn clean install
docker build -t networknt/com.networknt.apid-1.0.0 -f docker/Dockerfile .
```

## Start Services

Now that we have all four API built and dockerized, let's start the docker-compose.

{{% warning %}}
Before we start the new docker-compose, make sure you stop the current consul docker container.
{{% /warning %}}

Start Consul using docker-compose:

```bash
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-consul.yml up -d
```

Start APIs once Consul is ready. 

```bash
docker-compose -f docker-compose-discovery.yml up -d
```

Now you can go to the Consul Web UI to check if all APIs are registered. 

```bash
http://localhost:8500/
```

### Test Servers

```bash
curl -k https://localhost:7441/v1/data
```

And here is the result.

```json
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

In this step, we have dockerized all APIs and start them together. We also, start Consul with separate compose first to allow all services to register them to the Consul server. To make it simple, we are using HTTP to connect to the Consul and still using the static port in the server.yml config file. If you want to learn production like docker-compose and Consul integration please take a look at [Docker compose and Consul production][].

In the next step, we are going to deploy all services to [Kubernetes][] cluster instead of docker-compose. 

[Kubernetes]: /tutorial/common/discovery/kubernetes/
[docker host ip]: /tutorial/eventuate/getting-started/
[Docker compose and Consul production]: /tutorial/common/discovery/compose-consul/
