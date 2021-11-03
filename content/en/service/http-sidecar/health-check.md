---
title: "Health Check"
date: 2021-11-02T18:08:11-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When deploying the http-sidecar along with a backend API in the same pod in a Kubernetes cluster, we need to have two health check endpoints injected into the OpenAPI specification on the sidecar. And a special health check handler will be used instead of the standard [health check][] handler in light-4j. 


```
  /health/${server.serviceId}:
    get:
      description: The light-4j ProxyHealthGetHandler that is optionally invoke the backend API
      security:
        - admin-scope:
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
```

The first endpoint is used for the Kubernetes to probe the container without security and serviceId as the path parameter. Both liveness and readiness probes will invoke this endpoint. 

The control plane and third-party monitoring tools will invoke the second endpoint to ensure the API is healthy during the runtime. The OAuth 2.0 token protects this endpoint with proper scopes, and that is why we need to inject it into the OpenAPI specification during the server startup. It will enable the light-4j security handler to validate the token and scope for each request.

For the backend, you can implement /health or /health/liveness or /health/readiness endpoint, and it can be invoked by the health check handler from the http-sidecar. The path for the backend health check is configurable in the health.yml on the sidecar.

The same endpoint in the backend is not security protected, and Kubernetes will use the endpoint for the liveness and readiness probes. 


### Requirement

The following is a list of requirements that we need to support.

* Optionally invoke the backend health check endpoint. 
* Cache the connection to the backend to reduce the resource usage.
* Reset the connection cache after 1 million requests
* Work with Java EE server that open the socket before the server is ready. If connection is not working, close it.
* The health check handler should work for both containerized APIs or Lambda functions.

### Design

Since this handler depends on the Http2Client, we will put it into the ingress-proxy instead of the health as the health sub-project doesn't have a dependency on the client at the moment. 

It will reuse the same HealthConfig object for the configuration, and there is no need to add any configuration properties. 

During the pod startup phase, the light-4j http-sidecar can be started within a second, but some backend API will take up to 1 minute or more to start. During this period of time, the ProxyHealthGetHandler cannot establish a connection to the backend. It will return the error response immediately. And it won't try to send a request in this case as the connection is null. 

For Java EE backend started with liveness and readiness phases, we need to handle the request sending exception when the opened connection cannot send the request. If we have the exception, we need to close the connection so that a new connection will be recreated for the next health check invocation.


Even in normal runtime, there would be some cases that the request to the backend failed. In this condition, we need to close the connection to recreate a new connection for the next request. 

For some of the backend APIs that take over 2 minutes to start, the instance might be removed from the control plane as the default removal period is 2 minutes in the portal-registry.yml file. The project team can increase this value to 4 minutes or 5 minutes, depending on the startup time for the API.

```
# De-register the service after the amount of time with health check failed. Once a health check is failed, the
# service will be put into a critical state. After the deregisterAfter, the service will be removed from discovery.
# the value is an integer in milliseconds. 1000 means 1 second and default to 2 minutes
deregisterAfter: ${portalRegistry.deregisterAfter:120000}

```

[health check]: /concern/health/
