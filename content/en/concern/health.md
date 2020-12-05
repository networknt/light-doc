---
title: "Health Check"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This is a server health check handler that outputs OK or a JSON object to indicate the server is alive. Normally, it will be used by F5, light-router to check if the server is healthy before route requests to it. The global registry will also use the same endpoint to ensure all instances there are healthy for consumers to discover. Another way to check server health is to ping the IP and port, and it is the standard way to check server status for F5 or any reverse proxy servers. However, the service instance is up and running doesn't mean it is functioning. The handler can output more information about the server for reverse proxies. 

We recommend using the light-router as the router for Internet traffic for back-end services with static IP addresses that act as a traditional web server. These services will be sitting in DMZ to serve mobile native and browser SPA and aggregate other back-end services. There is no reverse proxy for services deployed in the cloud dynamically but using client-side service discovery.

This handler needs to be injected into the handler.yml as a standard path to return something that indicates the server is still alive. Currently, it returns "OK"  or {"result": "OK"} only, and in the future, it will be enhanced to add more features.



### Configuration


##### health.yml

Here is the default config health.yml for the module.

```
# Server health endpoint that can output OK to indicate the server is alive

# true to enable this middleware handler.
enabled: ${health.enabled:true}

# true to return Json format message
useJson: ${health.useJson:false}
```

##### handler.yml

As this endpoint is not part of each API specification, we need to inject it into the handler.yml file.

```
handlers:
  - com.networknt.health.HealthGetHandler@health


.
.
.

paths:
  - path: '/health/${server.serviceId:com.networknt.petstore-3.0.1}'
    method: 'get'
    exec:
      # - security
      - health

```

When you generate the project from light-codegen, the handler.yml will have an entry for the health check handler. Please note that the path for the health contains the serviceId as the path parameter. A service deployed to the cloud environment can crash in one host and restart in another host. Another service can immediately occupy the same IP and port. When the light-controller is trying to send the health check request, it doesn't know if the response is from the original service or a new service just occupied the IP and port. That is why we have the serviceId as the path parameter.

In the above example, I have commented out the security handler in the exec for the health. In a real service deployed to the cloud, this has to be enabled. The JwtVerifierHandler in light-rest-4j, light-hybrid-4j and light-graphql-4j can verify the token with the signature and check the scope for 'portal.w' and 'portal.r'. The light-controller is using a bootstrap token to connect to the endpoint for the health check with the proper scopes. 


### Proxy

When using light-proxy as a sidecar for backend API in a Kubernetes cluster, there are two options to config the health check on the light-proxy container. 

##### Proxy Health

If the backend API is implemented with other frameworks/languages, chances are it doesn't have a health check endpoint. In this case, we just need to set up the proxy to return its health check result on behalf of the backend API.

The configuration is the same as the above example. Just be sure to use the backend API serviceId for the path parameter. Only the light-proxy instance will be registered to the global registry for consumers to discover with this configuration. It is a standard way to bring any API service to the Light-4j ecosystem. 

##### Pass Through

If the backend API has the health check endpoint implemented, we want to call the backend API instead of the handler on the light-proxy. Here is the configuration in the handler.yml on the light-proxy.

```
handlers:
  - com.networknt.proxy.LightProxyHandler@proxy

  .
  .
  .

paths:
  - path: '/health/${server.serviceId:com.networknt.petstore-3.0.1}'
    method: 'get'
    exec:
      # - security
      - proxy

```
As you can see, we put the proxy handler in the exec for the /health/{serviceId} endpoint so the request will be proxied to the backend API. 

### Customization

The health check handler we implemented in the light-4j is just a reference implementation. I would recommend implementing customized handlers for real APIs to provide health info based on the dependencies. 

For example, if an API connects to a database to serve its request, it would check the database connection before returning the OK to the health check call. 

