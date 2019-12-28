---
title: "Proxy Server for Transactions"
date: 2019-12-23T11:27:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we have config the light-router instance to [route][] traffic to the microservices we have built. It is recommended that each application has a router instance or a cluster of instances. In this snese, the light-router is designed for consumers. 

For an enterprise, services might be built with different technology stack and it is not feasible to rewrite all of them in microservices. When you build a new application, chances are you need to leverage some of the legecy APIs. To bring these APIs to the light-4j ecosystem, we have introduced a gateway light-proxy to address all cross-cutting concerns. Unlike the light-router that is consumer focused, the light-proxy is provider focused for services not built on top of light-4j. For services built on top of light-4j, there is no need to use light-proxy as all the gateway cross-cutting concerns are embedded. 


To start using the light-proxy, let's create another folder under light-config-test named proxy and copy all the content from service folder to it. 

```
cd ~/open-banking/light-config-test
mkdir proxy
cp -r service/* proxy
```

Now, we have a duplication of the docker-compose.yml and all the configurations for four services. We need to make some modifications to disable the registry for transactions service so that it is not visiabl on the Consul. We also want to add a light-proxy service and config it to connect to the transaction. 

First let's update the configuration for the transactions service to ensure that it is not register itself to the Consul and bound to a specific port. 

values.yml

```
openapi-security.enableVerifyJwt: true
server.dynamicPort: false
server.enableRegistry: false
client.verifyHostname: false

```

As you can see, the registration is disabled for transactions service.


In the proxy configuration folder. We need to add the client.truststore with the certificate for the Consul cluster. We also need to add the client.yml, service.yml and consul.yml for Consul registry. 

As there is no default configuration available for the proxy server, we need to add server.yml and server.keystore as well. 

The values.yml as follows. 


```
# Use this config file to overwrite values from other config files.
server.serviceId: com.networknt.ob.transactions-3.1.2
server.enableRegistry: true
server.dynamicPort: true
client.verifyHostname: false
openapi-security.enableVerifyJwt: true
openapi-validator.skipBodyValidation: true
```

First, we overwrite the serviceId with the transactions serviceId so that other services will treat the proxy instance as the transactions service. Then enable the registry to the Consul. 

There is another important config file proxy.yml for the proxy server.

```
---
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: true

# If TLS is enabled when connecting to the target servers
httpsEnabled: true

# Target URIs
hosts: https://38.113.162.52:8443

# Connections per thread to the target servers
connectionsPerThread: 20

# Max request time in milliseconds before timeout
maxRequestTime: 1000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3

```

To eanble the validation and security, we need to copy the specification for the transactions to the config folder.

In the handler.yml file, we need to add the paths for the transactions service.

```
# Handler middleware chain configuration
---
enabled: true

#------------------------------------------------------------------------------
# Support individual handler chains for each separate endpoint. It allows framework
# handlers like health check, server info to bypass majority of the middleware handlers
# and allows mixing multiple frameworks like OpenAPI and GraphQL in the same instance.
#
# handlers  --  list of handlers to be used across chains in this microservice
#               including the routing handlers for ALL endpoints
#           --  format: fully qualified handler class name@optional:given name
# chains    --  allows forming of [1..N] chains, which could be wholly or
#               used to form handler chains for each endpoint
#               ex.: default chain below, reused partially across multiple endpoints
# paths     --  list all the paths to be used for routing within the microservice
#           ----  path: the URI for the endpoint (ex.: path: '/v1/pets')
#           ----  method: the operation in use (ex.: 'post')
#           ----  exec: handlers to be executed -- this element forms the list and
#                       the order of execution for the handlers
#
# IMPORTANT NOTES:
# - to avoid executing a handler, it has to be removed/commented out in the chain
#   or change the enabled:boolean to false for a middleware handler configuration.
# - all handlers, routing handler included, are to be listed in the execution chain
# - for consistency, give a name to each handler; it is easier to refer to a name
#   vs a fully qualified class name and is more elegant
# - you can list in chains the fully qualified handler class names, and avoid using the
#   handlers element altogether
#------------------------------------------------------------------------------
handlers:
  # Light-framework cross-cutting concerns implemented in the microservice
  - com.networknt.exception.ExceptionHandler@exception
  # - com.networknt.metrics.MetricsHandler@metrics
  - com.networknt.traceability.TraceabilityHandler@traceability
  - com.networknt.correlation.CorrelationHandler@correlation
  #Cors handler to handler post/put pre-flight
  # - com.networknt.cors.CorsHttpHandler@cors
  - com.networknt.openapi.OpenApiHandler@specification
  - com.networknt.openapi.JwtVerifyHandler@security
  # - com.networknt.body.BodyHandler@body
  # - com.networknt.audit.AuditHandler@audit
  # - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Header middleware to manipulate request and/or response headers before or after downstream server
  - com.networknt.header.HeaderHandler@header
  # Direct requests to named services based on the request path
  # - com.networknt.router.middleware.PathPrefixServiceHandler@path
  - com.networknt.proxy.LightProxyHandler@proxy
  # - com.networknt.resource.VirtualHostHandler@virtual
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus

chains:
  default:
    - exception
    # - metrics
    - traceability
    - correlation
    # - cors
    - header
    # - path
    - proxy
    - specification
    - security
    #- body
    #- audit
    #- sanitizer
    - validator

paths:
  - path: '/transactions/accounts/{AccountId}'
    method: 'GET'
    exec:
      - default
  - path: '/transactions'
    method: 'GET'
    exec:
      - default
  - path: '/health/com.networknt.ob.transactions-3.1.2'
    method: 'get'
    exec:
      - health

  # In most case, the /server/info endpoint shouldn't be exposed. If it is, then it must be protected by OAuth 2.0 or Basic Auth
  - path: '/server/info'
    method: 'get'
    exec:
      - info

```



[route]: /tutorial/open-banking/router/
