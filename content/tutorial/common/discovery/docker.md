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

In this step, we're going to dockerize all the APIs and then use [registrator][] for service 
registry. To make it easier, we are going to use docker-compose to put everything together.

First let's copy from consul to docker for each API.
 
```bash
cp -r ~/networknt/discovery/api_a/consul ~/networknt/discovery/api_a/consuldocker
cp -r ~/networknt/discovery/api_b/consul ~/networknt/discovery/api_b/consuldocker
cp -r ~/networknt/discovery/api_c/consul ~/networknt/discovery/api_c/consuldocker
cp -r ~/networknt/discovery/api_d/consul ~/networknt/discovery/api_d/consuldocker
```

## Configuring the APIs 

### API A

We will be running in a dockerized environment, so having consul configured at "localhost" is no longer valid.
Since we will be naming the container "consul", let's change the constructor of `ConsulClientImpl`:

`service.yml`

```yaml
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: consul
    - int: 8500
```

Since we are using registrator to register the service, we need to disable the application service registration:

`server.yml`

```yaml
enableRegistry: false
```

Please note that enableRegistry is set to false as we don't need the server to register itself
as it is running inside the Docker container and doesn't know the Container IP address and port.

The Dockerfile generated from light-codegen should expose `7441` based on the `server.yml`:

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
EXPOSE  7441
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

`service.yml`

```yaml
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: consul
    - int: 8500
```

`server.yml`

```yaml
enableRegistry: false
```

The Dockerfile generated from light-codegen should expose `7442` based on the `server.yml`:

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
EXPOSE  7442
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

`service.yml`

```yaml
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: consul
    - int: 8500
```

`server.yml`

```yaml
enableRegistry: false
```

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
EXPOSE  7443
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

`service.yml`

```yaml
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: consul
    - int: 8500
```

`server.yml`

```yaml
enableRegistry: false
```

`docker/Dockerfile`

```dockerfile
FROM openjdk:8-jre-alpine
EXPOSE 7444
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
git clone git@github.com:networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-consul.yml up -d
```

Start Registrator once Consul is ready.

```bash
docker-compose -f docker-compose-registrator.yml up -d
```

Start APIs once Registrator is ready.

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
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API C: Message 1","API C: Message 2"]
```

In this step, we have dockerized all APIs and start them together. We also, start Consul and Registrator
with separate compose and allow registrator to register all docker container on the host to Consul.

In the next step, we are going to deploy all services to [Kubernetes][] cluster instead of docker-compose. 

[registrator]: https://github.com/gliderlabs/registrator
[Kubernetes]: /tutorial/common/discovery/kubernetes/

