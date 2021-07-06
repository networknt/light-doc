---
title: "Virtual host on light-router"
date: 2018-11-02T23:08:37-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous tutorial, we have shown how to sever a single page application from a router instance. In this tutorial, we are going to expend it again to host multiple domains in the same router instance. 

The first step is to create a virtual-host folder in networknt/light-config-test/light-router and copy the content of taiji-faucet content into this folder. 

After that, we need to update the configuration files a little bit. 

### handler.yml

We need to update the handler.yml to add VirtualHostHandler inject to replace the PathResourceHandler.

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
  - com.networknt.resource.VirtualHostHandler@virtual
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
  - virtual

```


Please note that we have 

```
  - com.networknt.resource.VirtualHostHandler@virtual
```

And the defaultHandlers is defined as virtual

### virtual-host.yml

This file defines all the domains and their base. 

```
hosts:
  - domain: www.edibleforestgarden.ca
    path: /
    #base: /home/steve/networknt/site/edibleforestgarden/build
    base: /edibleforestgarden/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: taiji.io
    path: /
    #base: /home/steve/networknt/site/taiji/build
    base: /taiji/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: lightapi.net
    path: /
    #base: /home/steve/networknt/site/lightapi/build
    base: /lightapi/build
    transferMinSize: 10245760
    directoryListingEnabled: false

```

As this file is added, we need to remove the path-resource.yml as it is only for a single domain. Please note that we have the absolute path for base commented out. They are used when you are running the light-portal in a standalone mode or running it in an IDE for debugging. 

### Create Applications

Now, let's create several react applications.

```
cd ~/networknt/light-config-test/light-router/virtual-host
npx create-react-app edibleforestgarden
npx create-react-app lightapi
npx create-react-app taiji
```

You should see there are three folders created. The next step is to modify these application so that we can differenciate them by domain. 


Go to each folder and edit the App.js under src to replace

```
Edit <code>src/App.js</code> and save to reload.
```

To the corresponding domain name. 

```
Edible Forest Garden
```

```
Light API Portal
```

```
Taiji Blockchain
```

### Build Applications

Go to each SPA folder and run

```
yarn build
```

After that you can see there is build folder created under these three folders. You can also run the application to confirm that changes are shown on the home page. To start the server. 

```
yarn start
```

Repeat the above steps for all three domains. 


### DNS Setup

To support multiple hosts, we need to set up DNS to point these host to the server. If you are using real VM, then you need to update your DNS entry to point to that address for all three hosts.

In my local test, we are going to update the /etc/hosts with the following.

```
127.0.0.1       localhost www.edibleforestgarden.ca taiji.io lightapi.net
```

To ensure that DNS setup is working, you can ping these three sites. 


```
ping www.edibleforestgarden.ca
ping lightapi.net
ping taiji.io
```

### Update Compose

Now let's update the docker-compose.yml to add the volume mapping for the three domains. 

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
    - ./edibleforestgarden/build:/edibleforestgarden/build
    - ./lightapi/build:/lightapi/build
    - ./taiji/build:/taiji/build

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true

```

As you can see that we have mapped local SPAs build folders into the docker container folders defined in virtual-host.yml

### Start Compose

Now let's start the docker-compose and try it out.

Go to the browser and type the following addresses

```
https://www.edibleforestgarden.ca:8443
https://taiji.io:8443
https://lightapi.net:8443
```

And you can see the different home page per domain. 

### Debugging

If the home page is not shown up, then you can debug the router in your IDE to see what is really going on. 

Before debugging session you need to shutdown the docker-compose. 

```
docker-compose down
```

Also you need to update the virtual-host.yml to use the absolute path for base in each domain. 

Here is the updated file. 

```
hosts:
  - domain: www.edibleforestgarden.ca
    path: /
    base: /home/steve/networknt/light-config-test/light-router/virtual-host/edibleforestgarden/build
    #base: /edibleforestgarden/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: taiji.io
    path: /
    base: /home/steve/networknt/light-config-test/light-router/virtual-host/taiji/build
    #base: /taiji/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/virtual-host/lightapi/build
    #base: /lightapi/build
    transferMinSize: 10245760
    directoryListingEnabled: false

```


Now we need to set up the debugging session IntelliJ IDEA for the externalized config folder as below. 

```
-Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-router/virtual-host/config
```

If you are unsure how to debug light-4j application, please refer to this [video](https://www.youtube.com/watch?v=qW0xIC_A_KI&t=50s)


### Video

Here is a video that walks through the configuration and gives a demo. 

{{< youtube fYr3GmunLMU >}}



