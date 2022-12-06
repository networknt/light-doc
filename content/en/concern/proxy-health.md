---
title: "Proxy Health"
date: 2022-11-30T10:59:21-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For an API built with light-4j frameworks, it has a [health][] check endpoint exposed for third-party monitoring systems and Kubernetes probes. If http-sidecar is used, we can use the LightProxyHandler for the health check, and it has the option to collect the health status from the backend API. 

Both HealthGetHandler and ProxyHealthGetHandler share the same health.yml configuration file. 

In the handler.yml, we can also config two different endpoints. 

```
handlers:
  - com.networknt.proxy.ProxyHealthGetHandler@health
.
.
.

paths:
  - path: '/adm/health/${server.serviceId:com.networknt.petstore-3.0.1}'
    method: 'get'
    exec:
      - security
      - health

  - path: '/health
    method: 'get'
    exec:
      - health

```

Some users may prefer K8s probes to use /health with the HealthGetHandler and others may prefer /adm/health/${server.serviceId:} to use the ProxyHealthGetHandler. 


##### Proxy Health

If the backend API is implemented with other frameworks/languages, chances are it doesn't have a health check endpoint. In this case, we just need to set up the proxy to return its health check result on behalf of the backend API.

The configuration is the same as the above example. Just be sure to use the backend API serviceId for the path parameter. Only the http-sidecar or light-gateway instance will be registered to the global registry for consumers to discover with this configuration. It is a standard way to bring any API service to the Light-4j ecosystem. 

##### Pass Through

If the backend API has the health check endpoint implemented, we want to call the backend API instead of the handler on the http-sidecar or light-gateway. Here is the configuration in the handler.yml on the http-sidecar or light-gateway.

```
handlers:
  - com.networknt.proxy.LightProxyHandler@proxy

  .
  .
  .

paths:
  - path: '/adm/health/${server.serviceId:com.networknt.petstore-3.0.1}'
    method: 'get'
    exec:
      # - security
      - proxy

```
As you can see, we put the proxy handler in the exec for the /adm/health/{serviceId} endpoint so the request will be proxied to the backend API. 

##### Aggregated

When used in the http-sidecar or light-gateway, there is another option to combine both the gateway and backend health statuses together. 

You need to use ProxyHealthGetHandler and overwrite the health.yml configuration in the values.yml file.


```
health.downstreamEnabled: true
health.downstreamHost: http://localhost:8081
health.downstreamPath: /health
```

First, we enable the invocation to the backend API. Then we need to set up the backend API host and path for the health check endpoint. 


### Customization

The health check handler we implemented in the light-4j is just a reference implementation. I would recommend implementing customized handlers for real APIs to provide health info based on the dependencies. 

For example, if an API connects to a database to serve its request, it would check the database connection before returning the OK to the health check call. 

One example of a health check is the Kafka sidecar that checks the backend connectivity when Reactive Consumer is enabled. The source code can be accessed at https://github.com/networknt/kafka-sidecar/blob/master/src/main/java/com/networknt/mesh/kafka/handler/SidecarHealthHandler.java



[health]: /concern/health/
