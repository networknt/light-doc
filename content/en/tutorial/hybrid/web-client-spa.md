---
title: "Web Client SPA"
date: 2018-12-03T08:29:42-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Since we have introduced handler.yml for multi-chain support in the light-4j platform, there are a lot of enhancements added to these frameworks. One of the most important features is to support Single Page Application with path-resource and virtual-host. 

In this tutorial, we are going to reconfigure the generated light-hybrid-4j server to serve SPA with handler.yml file. The project is the web-client for taiji-blockchain and all source code can be accessed from the [github repo](https://github.com/taiji-chain/web-client). The sub-project we are going to modify is the server. 

As of release 1.5.23, the light-hybrid-4j codegen still using PathHandlerProvider to define the handlers chain. To support path-resource handler, we are going to make some configuration change to enable handler.yml

First, let's create a new folder under server called config. This folder will contain different configuration files for local development, testnet, and mainnet. For this tutorial, we are going to use a local folder for the configuration. 

### Copy config

Create a local folder under config and copy all the configuration files from the default folder under src/test/resources/config.

### handler.yml

Add a handler.yml file with the content like below. 

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
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  # Cors handler to handler post/put pre-flight
  # - com.networknt.cors.CorsHttpHandler@cors
  # - com.networknt.openapi.OpenApiHandler@specification
  # - com.networknt.openapi.JwtVerifyHandler@security
  # - com.networknt.body.BodyHandler@body
  # - com.networknt.audit.AuditHandler@audit
  # - com.networknt.sanitizer.SanitizerHandler@sanitizer
  # - com.networknt.openapi.ValidatorHandler@validator
  # Header middleware to manipulate request and/or response headers before or after downstream server
  # - com.networknt.header.HeaderHandler@header
  # Direct requests to named services based on the request path
  # - com.networknt.router.middleware.PathPrefixServiceHandler@path
  # - com.networknt.router.RouterHandler@router
  - com.networknt.resource.PathResourceHandler@resource
  - com.networknt.rpc.router.JsonHandler@json
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus

chains:
  default:
    - exception
    - json

paths:
  - path: '/api/json'
    method: 'GET'
    exec:
      - default
  - path: '/api/json'
    method: 'POST'
    exec:
      - default

  - path: '/health/io.taiji.server-1.0.0'
    method: 'get'
    exec:
      - health

  # In most case, the /server/info endpoint shouldn't be exposed. If it is, then it must be protected by OAuth 2.0 or Basic Auth
  - path: '/server/info'
    method: 'get'
    exec:
      - info


defaultHandlers:
  - resource

```

To make it simple, we only enable the exception handler and JsonHandler as default. For the JsonHandler, it maps to the path `/api/json` for both GET and POST. 

Also, there are /health and server/info path and handlers wired in. For the SPA, we have added the defaultHandlers with the only path-resource handler. 

### path-resource.yml

We need to add this config file for the SPA resource handler. 

```
path: /
# This is the base used by docker
# base: /public
# This is the base that is used for IDE debugging
base: /home/steve/taiji-chain/web-client/server/view/build
prefix: true
transferMinSize: 10485760
directoryListingEnabled: false
```
As we are using IDE for testing and debugging, we need to use the absolute path on my desktop computer for the base. Once it is dockerized, you can use /public which will be mapped to a volume on a directory of the host. 

### service.yml

The generated service.yml contains PathHandlerProvider and all the middleware handlers for the chain. We need to remove them all as we have switched to handler.yml for the chain definition. After modification, the config file looks like this. 

```

# Singleton service factory configuration/IoC injection
singletons:
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  # registry all service handlers by from annotations
  - com.networknt.rpc.router.RpcStartupHookProvider

```

### SPA

Now, go to the root folder of server project and create a React app with the create-react-app tool and build it.

```
npx create-react-app view
cd view
yarn build
```
A build folder will be created with yarn build, and it contains the final SPA application ready for production. 


### pom.xml

There are two modules need to be added to the generated pom.xml as there are not included by default. 

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>resource</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>health</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```


### Summary


With the above steps, the server project is ready to serve static content. Start the server from IDE with the following VM options. 

```
-Dlight-4j-config-dir=/home/steve/taiji-chain/web-client/server/config/local
```

And go to your browser with the URL https://localhost:8443 and you can see the React application running. 


