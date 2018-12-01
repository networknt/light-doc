---
title: "Schema Registry with Proxy"
date: 2018-11-30T22:20:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When building the taiji-blockchain, we are using Avro for binary encoding and decoding for service to service communication. To make sure that the size of the binary data is smallest, we need a centralized server to save all the schema so that only a schema ID will be serialized into the binary data. Both clients and services will go to the centralized schema registry to get the Avro schema to encode and decode the data. 

We are using the Confluent schema-registry docker image for this purpose; however, we found that there is no authorization built-in and a plugin is available for enterprise version which needs a commercial license. 

To add a security layer to allow Internet users/apps to access read-only endpoints, we are going to deploy an instance of light-proxy in front of schema-registry. 


### Install Schema Registry

It is very easy to start the schema-registry with docker. 

```
docker run -d \
-e SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=38.113.162.53:2181 \
-e SCHEMA_REGISTRY_HOST_NAME=schema-registry \
-e SCHEMA_REGISTRY_LISTENERS=http://0.0.0.0:8081 \
-p 8081:8081 \
confluentinc/cp-schema-registry:5.0.1
```

As you can see that the schema-registry is listening to port 8081. We have a firewall on the server to block the access to this port so nobody can access this server except a client from the localhost. To allow internal applications to access this service directly, we are going to open the firewall for several IP addresses. 

To confirm the server is working, issue the following curl command. 

```
curl http://localhost:8081/subjects
```

And an empty array should be returned if the server is not populated yet. 

### Light proxy

First, let's create a config folder in light-config-test repo for externalized config files. 

```
cd ~/networknt/light-config-test
mkdir light-proxy
cd light-proxy
mkdir schema-registry
```

Now, let's create config files. 

##### proxy.yml

We need to have proxy.yml to config the downstream service and other parameters. 


```
---
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: false

# If TLS is enabled when connecting to the target servers
httpsEnabled: false

# Target URIs
hosts: http://38.113.162.50:8081

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

There is only one target host which is http://38.113.162.50:8081. Please note that we are not using localhost but real IP address for the downstream service as we want to run the light-proxy in docker as well. Within the docker contain, localhost means itself, but the schema registry is running on the DOCKER_HOST. If you are planning to use docker-compose, you can change the IP address to the container name. 


##### server.yml

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  8080

# Enable HTTP should be false on official environment.
enableHttp: true

# Https port if enableHttps is true.
httpsPort:  8080

# Enable HTTPS should be true on official environment.
enableHttps: false

# Http/2 is enabled. When Http2 is enable, enableHttps is true and enableHttp is false by default.
# If you want to have http enabled, enableHttp2 must be false.
enableHttp2: false

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.proxy-0.1.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

```

Only the HTTP port 8080 is opened on the light-proxy.

##### handler.yml

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
  #Cors handler to handler post/put pre-flight
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
    #- metrics
    # - traceability
    # - correlation
    # - cors
    # - header
    # - path
    - proxy
    #- specification
    #- security
    #- body
    #- audit
    #- sanitizer
    #- validator

paths:
  - path: '/config'
    method: 'GET'
    exec:
      - default
  - path: '/subjects'
    method: 'GET'
    exec:
      - default
  - path: '/schemas/ids/{id}'
    method: 'GET'
    exec:
      - default
  - path: '/subjects/{subject}/versions'
    method: 'GET'
    exec:
      - default
  - path: '/subjects/{subject}/versions/{version}'
    method: 'GET'
    exec:
      - default
  - path: '/subjects/{subject}/versions/{version}/schema'
    method: 'GET'
    exec:
      - default
  - path: '/config/{subject}'
    method: 'GET'
    exec:
      - default

  - path: '/health/com.networknt.proxy-0.1.0'
    method: 'get'
    exec:
      - health

  # In most case, the /server/info endpoint shouldn't be exposed. If it is, then it must be protected by OAuth 2.0 or Basic Auth
  - path: '/server/info'
    method: 'get'
    exec:
      - info
```

In the middleware chain, only the exception and proxy handlers are wired in as default. And all the read-only paths are defined here. 

### Start the proxy

Log into the target server which has the schema registry deployed and clone the light-config-test repo to it in the networknt folder. 

```
cd ~/networknt
git clone https://github.com/networknt/light-config-test.git
cd light-config-test/light-proxy
```

In this directory, there is a schema-registry folder contains all the config files for the schema-registry proxy. 


To start the light-proxy with Docker

```
docker run -d \
-v $(pwd)/schema-registry:/config \
-p 8080:8080 \
networknt/light-proxy
```

Or it is better to see the output on the console the first time. 

```
docker run \
-v $(pwd)/schema-registry:/config \
-p 8080:8080 \
networknt/light-proxy

```


### Test the proxy

You can use the following command to test the proxy server. 

```
curl http://localhost:8080/subjects
```

### Docker compose

To start docker container separately gives us the flexibility but it takes a lot of efforts to maintain several containers. To make it easier, let's create a docker-compose file so that we can start the schema-registry and light-proxy at the same time. 

First, let's create a folder in light-docker repo. 


```
cd ~/networknt/light-docker/light-proxy
mkdir schema-registry
```

Create a docker-compose-schema-registry.yml file in the root folder of the light-docker. 

```
version: "2"
#
# Services
#
services:
  schema-registry:
    image: confluentinc/cp-schema-registry:5.0.1
    ports:
      - 8081:8081
    environment:
      - SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=38.113.162.53:2181
      - SCHEMA_REGISTRY_HOST_NAME=schema-registry
      - SCHEMA_REGISTRY_LISTENERS=http://0.0.0.0:8081
    networks:
      - localnet

  light-proxy:
    image: networknt/light-proxy:latest
    ports:
      - 8080:8080
    volumes:
      - ./light-proxy/schema-registry:/config
    links:
      - schema-registry
    networks:
      - localnet

# Networks
#
networks:
  localnet:
    external: true

```

Copy the three config files to the schema-registry folder. As we are using docker-compose, we need to update the IP to the docker-compose service name. 

```
---
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: false

# If TLS is enabled when connecting to the target servers
httpsEnabled: false

# Target URIs
hosts: http://schema-registry:8081

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

Note that we change the IP to service name schema-registry. 

On the target server, make sure we stop both the light-proxy and the schema-registry containers. 

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-schema-registry.yml up -d
```

To test the proxy and registry services. 

```
curl http://localhost:8081/subjects
curl http://localhost:8080/subjects
```

Both commands should have the same result. An empty array should be returned if this is a newly installed. 


### Summary

With this setup, we have the original schema-registry service running on port 8081 and the light-proxy server running on port 8080. On the target server, we will open the port 8080 to the Internet so everyone can access the read-only endpoints. 

```
sudo ufw allow 8080
```

For your internal services to access the original schema-registry port 8081, you can add ufw rule to do that. 

To allow my home computer to access the original service.

```
sudo ufw allow from xxx.xxx.xxx.xxx to any port 8081
```

### Resource

This tutorial is based on a real configuration on taiji-blockchain testnet. You can access the real server with the following URL.

```
curl http://38.113.162.50:8080/subjects
```
For the docker configuration, you can find it at https://github.com/networknt/light-config-test/tree/master/light-proxy/schema-registry

For the docker-compose file and configuration, you can find them at https://github.com/networknt/light-docker/tree/master/light-proxy/schema-registry


