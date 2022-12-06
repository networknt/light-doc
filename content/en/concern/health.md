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

This is a server health check handler that outputs OK or a JSON object to indicate the server is alive. Normally, it will be used by F5/light-gateway/http-sidecar to check if the server is healthy before routing requests to it. The global registry will also use the same handler to ensure all instances are healthy for consumers to discover. Another way to check server health is to ping the IP and port, and it is the standard way to check server status for F5 or any reverse proxy servers. However, just because the service instance is up and running doesn’t mean it is functioning. The handler can output more information about the server for reverse proxies.

We recommend using the light-gateway as the router for Internet traffic for back-end services with static IP addresses that act as a traditional web server. These services will be sitting in DMZ to serve mobile native and browser SPA and aggregate other back-end services. There is no reverse proxy for services deployed in the cloud dynamically but using client-side service discovery.

This handler needs to be injected into the handler.yml as a standard path to return something that indicates the server is still alive. Currently, it returns "OK"  or {"result": "OK"} only, and in the future, it will be enhanced to add more features.

When deploying the server to a Kubernetes cluster, there are three possible ways the health check endpoint is invoked:

1. Kubernetes liveness and readiness probes
2. Light Control Pane
3. Third-party monitor tools.


### Configuration


##### health.yml

Here is the default config health.yml for the module.

```
# Server health endpoint that can output OK to indicate the server is alive

# true to enable this middleware handler. By default the health check is enabled.
enabled: ${health.enabled:true}
# true to return Json format message. By default, it is false. It will only be changed to true if the monitor
# tool only support JSON response body.
useJson: ${health.useJson:false}
# timeout in milliseconds for the health check. If the duration is passed, a failure will return.
# It is to prevent taking too much time to check subsystems that are not available or timeout.
# As the health check is used by the control plane for service discovery, by default, one request
# per ten seconds. The quick response time is very important to not block the control plane.
timeout: ${health.timeout:500}

# For some of the services like light-proxy, http-sidecar and kafka-sidecar, we might need to check the down
# stream API before return the health status to the invoker. By default it is not enabled.

# if the health check needs to invoke down streams API. It is false by default.
downstreamEnabled: ${health.downstreamEnabled:false}
# down stream API host. http://localhost is the default when used with http-sidecar and kafka-sidecar.
downstreamHost: ${health.downstreamHost:http://localhost}
# down stream API health check path. This allows the down stream API to have customized path implemented.
downstreamPath: ${health.downstreamPath:/health}
```

##### handler.yml

As this endpoint is not part of each API business specification, we need to inject it into the handler.yml file.

```
handlers:
  - com.networknt.health.HealthGetHandler@health


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

When you generate the project from light-codegen, the handler.yml will have an entry for the health check handler. Please note that the path for the health contains the serviceId as the path parameter. A service deployed to the cloud environment can crash in one host and restart in another host. Another service can immediately occupy the same IP and port. When the light-controller is trying to send the health check request, it doesn't know if the response is from the original service or a new service just occupied the IP and port. That is why we have the serviceId as the path parameter.

In the above example, I have commented out the security handler in the exec for the health. In a real service deployed to the cloud, this has to be enabled. The JwtVerifierHandler in light-rest-4j, light-hybrid-4j and light-graphql-4j can verify the token with the signature and check the scope for 'portal.w' and 'portal.r'. The light-controller is using a bootstrap token to connect to the endpoint for the health check with the proper scopes. 

When a service is deployed to a Kubernetes cluster, we might need to add a new endpoint /health without security. The K8s will use this for liveness and readiness probes. 

### Proxy

For proxy health check, please refer to [proxy health][]

### Customization

The health check handler we implemented in the light-4j is just a reference implementation. I would recommend implementing customized handlers for real APIs to provide health info based on the dependencies. 

For example, if an API connects to a database to serve its request, it would check the database connection before returning the OK to the health check call. 

One example of a health check is the Kafka sidecar that checks the backend connectivity when Reactive Consumer is enabled. The source code can be accessed at https://github.com/networknt/kafka-sidecar/blob/master/src/main/java/com/networknt/mesh/kafka/handler/SidecarHealthHandler.java


[proxy health]: /concerns/proxy-health/
