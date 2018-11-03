---
title: "Router to render SPA"
date: 2018-11-02T23:08:23-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This tutorial is based on the taiji-faucet application in the folder networknt/light-config-test/light-router. The configuration is copied from taiji-faucet and modified in the router-spa folder. 

In the previous tutorial, we have shown you how to [create an instance of light-router for taiji-faucet service][] running in the Kubernetes cluster. 

In this tutorial, we are going to enable the light-router to serve a single page application built on React. I am not going to complete the SPA but show you the steps to get it set up. 

### handler.yml

To serve the static content from light-4j application, we need to change the configuration for light-router to handler.yml in order to wire in PathResourceHandler. 

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
  # - com.networknt.openapi.OpenApiHandler@specification
  # - com.networknt.openapi.JwtVerifyHandler@security
  # - com.networknt.body.BodyHandler@body
  # - com.networknt.audit.AuditHandler@audit
  # - com.networknt.sanitizer.SanitizerHandler@sanitizer
  # - com.networknt.openapi.ValidatorHandler@validator
  # Header middleware to manipulate request and/or response headers before or after downstream server
  - com.networknt.header.HeaderHandler@header
  # Direct requests to named services based on the request path
  - com.networknt.router.middleware.PathPrefixServiceHandler@path
  - com.networknt.router.RouterHandler@router
  - com.networknt.resource.PathResourceHandler@resource
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus

chains:
  default:
    - exception
    #- metrics
    - traceability
    - correlation
    - header
    - path
    - router
    #- specification
    #- security
    #- body
    #- audit
    #- sanitizer
    #- validator

paths:
  - path: '/faucet/{address}'
    method: 'GET'
    exec:
      - default
  - path: '/faucet/{address}'
    method: 'POST'
    exec:
      - default

  - path: '/health/com.networknt.router-0.1.0'
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

Please note that we have the 

```
  - com.networknt.resource.PathResourceHandler@resource
```

defined in the handlers section and this handler is referred in the defaultHandlers section at the end of the file.

```
defaultHandlers:
  - resource

```

This configuration tells the light-4j to handler all the other path/method combination with resource handler if they are not defined in the `paths`.

### path-resource.yml

We also need to define some configuration parameters for the PathResourceHandler. 

```
path: /
# This is the base used by docker
base: /public
# This is the base that is used for IDE debugging
#base: /home/steve/networknt/light-config-test/light-router/taiji-faucet/view/build
prefix: true
transferMinSize: 10485760
directoryListingEnabled: false
```

Please note that the configuration now is suitable for compose as a path in container is used. When you are running with debugger in an IDE, you need to switch to `base: /home/steve/networknt/light-config-test/light-router/taiji-faucet/view/`

This configuration tells the system to serve files from the base when there is request to the root path `/`. For example, if you put the following address in the address bar on your browser. 

```
https://localhost:8443
```

### Create application

Go the networknt/light-config-test/light-router/taiji-faucet folder and run the following command to create a React application called view.

```
npx create-react-app view
cd view
yarn build
```

Now we have a new React single page application built and it is readily to be 
served from the view/build folder. 

### Update Compose

Now let's update the compose file to map this folder into the container. 

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    networks:
    - localnet
    ports:
    - 8443:8443
    volumes:
    - ./config:/config
    - ./view/build:/public

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true

```

You can see that we have mapped ./view/build to /public inside the container. 


### Start Compose

Let's start the compose

```
docker-compose up
```

And then put the following URL into the address bar of your browser. 

```
https://localhost:8443/
```

You will see an React starter application running. The next step is to update the React applicationt to access APIs on the portal. 

[create an instance of light-router for taiji-faucet service]: /tutorial/router/taiji-faucet/


