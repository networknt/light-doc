---
title: "Portal Registry"
date: 2020-11-20T21:58:47-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Until recently, we always recommend our customers to use Consul for the global registry and discovery. However, we found some issues with Consul reported from our customers. So we started to work on our own registry on the portal to replace the Consul. For more information about this component, please refer to [Portal Registry][]. 

To make it simple, we are going to copy the four projects from consul-tls folders. 

```
cd api_a
cp -r consul-tls portal-registry
cd ../api_b
cp -r consul-tls portal-registry
cd ../api_c
cp -r consul-tls portal-registry
cd ../api_d
cp -r consul-tls portal-registry
```

### pom.xml

Update the pom.xml to replace consul with portal-registry. 

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>portal-registry</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```


### Add portal-registry.yml

To connect to the portal-registry, we need to replace the consul.yml with portal-registry.yml in all four projects.

```
# Portal URL for accessing controller API. Default to lightapi.net public portal and it can be point to a standalone
# light-controller instance for testing in the same Kubernetes cluster or docker-compose.
portalUrl: ${portalRegistry.portalUrl:https://localhost:8438}
# number of requests before resetting the shared connection to work around HTTP/2 limitation
maxReqPerConn: ${portalRegistry.maxReqPerConn:1000000}
# De-register the service after the amount of time with health check failed. Once a health check is failed, the
# service will be put into a critical state. After the deregisterAfter, the service will be removed from discovery.
# the value is an integer in milliseconds. 1000 means 1 second and default to 2 minutes
deregisterAfter: ${portalRegistry.deregisterAfter:120000}
# health check interval for HTTP check. Or it will be the TTL for TTL check. Every 10 seconds, an HTTP check
# request will be sent from the light-portal controller. Or if there is no heartbeat TTL request from service
# after 10 seconds, then mark the service is critical. The value is an integer in milliseconds
checkInterval: ${portalRegistry.checkInterval:10000}
# enable health check HTTP. An HTTP get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on the database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if DB is down, return status 400. This
# is the recommended configuration that allows the light-portal controller to poll the health info from each service.
httpCheck: ${portalRegistry.httpCheck:true}
# enable health check TTL. When this is enabled, The light-portal controller won't actively check your service to
# ensure it is healthy, but your service will call check endpoint with a heartbeat to indicate it is alive. This
# requires that the service is built on top of light-4j, and the HTTP check is not available. For example, your service
# is behind NAT. If you are running the service within your internal network and using the SaaS lightapi.net portal,
# this is the only option as our portal controller cannot access your internal service to perform a health check.
# We recommend deploying light-portal internally if you are running services within an internal network for efficiency.
ttlCheck: ${portalRegistry.ttlCheck:false}

```

### Update service.yml

Change the registry to PortalRegistry. 

```
# Singleton service factory configuration/IoC injection
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: portal
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.portal.registry.client.PortalRegistryClient:
  - com.networknt.portal.registry.client.PortalRegistryClientImpl
- com.networknt.registry.Registry:
  - com.networknt.portal.registry.PortalRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

### Update server.yml

In the server.yml, update the enableRegistry to true. Or add an entry in the values.yml 

```
server.enableRegistry: true
```

### Add client.yml and client.truststore

You need the light-4j Http2Client to connect to the light-controller.

Add client.yml if it is missing and corresponding cleint.truststore that contains the certificate issued by the light-controller.


### Start the light-controller

It is not an open-source project to the public but only to the paid customers. Users who want to try it can use the published docker image to start the service locally as a standalone demo. The enterprise version is part of the portal, and it can be accessed in the cloud from lightapi.net. 

To start the standalone version of light-controller. 

```
docker run -p 8438:8438 networknt/light-controller
```

### Start APIs

Start four services one by one. 

```
cd ~/networknt/light-example-4j/discovery/api_d
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar 
```

All servers are using dynamic port and register to the controller. 

### Controller Query

When the security is turned off, you can request the light-controller to show how many instances are registered. 


```
curl -k https://localhost:8438/services
```

Result after formatted.

```
{
  "com.networknt.ac-1.0.0": [
    {
      "protocol": "https",
      "address": "172.18.0.1",
      "port": 2460
    }
  ],
  "com.networknt.aa-1.0.0": [
    {
      "protocol": "https",
      "address": "172.18.0.1",
      "port": 2477
    }
  ],
  "com.networknt.ad-1.0.0": [
    {
      "protocol": "https",
      "address": "172.18.0.1",
      "port": 2474
    }
  ],
  "com.networknt.ab-1.0.0": [
    {
      "protocol": "https",
      "address": "172.18.0.1",
      "port": 2432
    }
  ]
}```

You can shut down one or more servers to check if they are removed from the above query.

### Discovery

In order to test the discovery, we need to start the multiple instances of the services with a docker-compose. If you have a Kubenetes cluster, it is even better. 

Here is the compose file in the light-docker repository. Please note that we have a values.yml in the portal-registry folder in the light-docker to specify the light-controller portalUrl with my local IP address. If you want to run this tutorial, you need to change these values.yml files with your local IP address.

docker-compose-portal-registry.yml

```
version: '2'

services:

  apia:
    image: networknt/apia-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apia:/config
    network_mode: host

  apib1:
    image: networknt/apib-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apib:/config
    network_mode: host

  apib2:
    image: networknt/apib-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apib:/config
    network_mode: host

  apic1:
    image: networknt/apic-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apic:/config
    network_mode: host

  apic2:
    image: networknt/apic-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apic:/config
    network_mode: host

  apid:
    image: networknt/apid-pr
    environment:
    - STATUS_HOST_IP=${DOCKER_HOST_IP}
    volumes:
    - ./portal-registry/apid:/config
    network_mode: host

```

Make sure you have the light-controller docker started before you start the docker-compose. 

```
docker-compose -f docker-compose-portal-registry.yml up
```

After all services are up and running with the docker-compose, you should see them on the light-controller console.

![light-controller](/images/light-controller.png)

Send several request to the apia_pr based on the ip and port from the above controller console.

```
curl -k https://localhost:2404/v1/data
```




[Portal Registry]: /concern/portal-registry/



