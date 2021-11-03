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
      description: The light-4j ProxyHealthCheckHandler that is optionally invoke the backend API
      security:
        - admin-scope:
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
```

The OAuth 2.0 token protects this endpoint with proper scopes, and that is why we need to inject it into the OpenAPI specification during the server startup. It will enable the light-4j security handler to validate the token and scope for each request.

For the backend, you can implement /health or /health/liveness or /health/readiness endpoint, and it can be invoked by the health check handler from the http-sidecar. 

The same endpoint in the backend is not security protected, and Kubernetes will use the endpoint for the liveness and readiness probes. 


### Requirement

The following is a list of requirements that we need to support.

* Optionally invoke the backend health check endpoint. 
* Cache the connection to the backend to reduce the resource usage.
* Reset the connection cache after 1 million requests
* Work with Java EE server that open the socket before the server is ready.




[health check]: /concern/health/
