---
title: "Portal Registry"
date: 2020-11-16T18:45:07-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Most of our users are using Consul for the service registry and discovery in large scale microservices deployment. However, we found several issues with Consul and finally released our light-portal controller to replace Consul as a centralized control panel.

The first issue with Consul is the local agent resource utilization and extra setup. If all services can register themselves to a centralized cluster, it would be more efficient. 

The second issue with Consul is the long poll or blocking query when API subscribes to the downstream API instance changes. It utilizes a lot of threads that are very valuable in the cloud environment. We need a more efficient way like WebSocket for the subscription updates.

The third issue is due to the software procurement cycle for our enterprise users. It takes our customers at least six months to one year to adopt the Consul, and it might be easier for them if we can provide something within the light-platform. 

Given the above reasons, we have developed a light-controller as the runtime microservice control panel for the light-portal. This light-4j module is used to connect to the light-controller. 

The implementation is similar to the Consul implementation but much more straightforward. For APIs that want to leverage the light-controller, we need to make the following changes. 


### pom.xml
In the generated pom.xml we need to replace the consul module with portal-registry. In the future, we might make the portal-registry the default. 

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>portal-registry</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

### server.yml

You can either change the server.yml file to enableRegistry to true or update the values.yml file to overwrite the server.enableRegistry to true.

```
enableRegistry: ${server.enableRegistry:true}
```

### service.yml

In this file, we need to make the change to use the portal-registry. 

```
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

### portal-registry

We also need to add a new config file called portal-registry.yml for this module.

```
# Portal URL for accessing controller API. Default to lightapi.net public portal and it can be point to a standalone
# light-controller instance for testing in the same Kubernetes cluster or docker-compose.
portalUrl: ${portalRegistry.portalUrl:https://localhost:8443}
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
httpCheck: ${portalRegistry.httpCheck:false}
# enable health check TTL. When this is enabled, The light-portal controller won't actively check your service to
# ensure it is healthy, but your service will call check endpoint with a heartbeat to indicate it is alive. This
# requires that the service is built on top of light-4j, and the HTTP check is not available. For example, your service
# is behind NAT. If you are running the service within your internal network and using the SaaS lightapi.net portal,
# this is the only option as our portal controller cannot access your internal service to perform a health check.
# We recommend deploying light-portal internally if you are running services within an internal network for efficiency.
ttlCheck: ${portalRegistry.ttlCheck:true}
# The health check path implemented on the server. In most of the cases, it would be /health/ plus the serviceId;
# however, on a kubernetes cluster, it might be /health/liveness/ in order to differentiate from the /health/readiness/
# Note that we need to provide the leading and trailing slash in the path definition.
healthPath: ${portalRegistry.healthPath:/health/}

```

### Flow

##### Registry

Suppose you are working on a service that is not consuming any other service. In that case, you only need to register during the service startup and remove the registry from the controller during the shutdown. 

The above actions will ensure that your consumers will discover instances at any time in the service's life-cycle. 

##### Discovery

For light-router or any service that consumers other services, we need to go to the light-controller to query the target service nodes based on the serviceId and an environment tag for the first invocation. After the initial query, the portal-registry will subscribe to the service changes so that the internal cache can be updated in realtime through WebSocket calls.



