---
title: "Service Discovery"
date: 2019-06-18T09:37:30-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we have instrumented the OpenAPI [petstore][] application and walked through the Jaeger UI. When talking about distributed tracing, we need to pass the context from one service to another service. If you remember, we have the [service discovery][] tutorial with four services calling each other. We are going to extend it with OpenTracing so that users can see how the original request flows through multiple services. We can also add a standalone client to invoke the api_a and pass in a tracing context. 

The four services for service discovery are located in the light-example-4j repository. If you haven't checked it out, please do so. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
```

Let's first make a copy for these services so that we can make changes. In the service discovery tutorial, the last step that modifies code is in the security folder. The OpenTracing is depending on the security to be enabled so that the headers can be propagated from service to service. 


```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r security open-tracing
cd ~/networknt/light-example-4j/discovery/api_b
cp -r security open-tracing
cd ~/networknt/light-example-4j/discovery/api_c
cp -r security open-tracing
cd ~/networknt/light-example-4j/discovery/api_d
cp -r security open-tracing
```

### Consul

To make sure that users can follow the tutorial locally, we are going to start the consul server locally with docker. 

```
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

### light-oauth2

We need to have the light-oauth2 services in order to get the client-credentials token for service to service calls. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up -d
```

For more details, please follow the security tutorial before starting this one. In that case, all light-oauth2 registration should be done already and both Consul and light-oauth2 should be up and running by now. 

### API D

Let's update API D 

First we need to update pom.xml to add the dependency of jaeger-tracing. 

pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

In the handler.yml file, comment out the traceability and correlation and add the jaeger. We put the JaegerHandler after the specification as it depends on it to use the OpenAPI path for the tracer operation.

```
---
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
  - com.networknt.metrics.MetricsHandler@metrics
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  # Open Tracing Jaeger Handler
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.ad.handler.DataGetHandler


chains:
  default:
    - exception
    - metrics
    # - traceability
    # - correlation
    - specification
    - jaeger
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/v1/data'
    method: 'GET'
    exec:
      - default
      - com.networknt.ad.handler.DataGetHandler

  - path: '/health/com.networknt.ad-1.0.0'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```

Also, we need to update the service.yml to add JaegerStartupHookProvider. 

```
  - com.networknt.server.StartupHookProvider:
      - com.networknt.jaeger.tracing.JaegerStartupHookProvider
```

Finally, we need to add jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

To make sure that we can use the long-lived token to access API D, we need to change the openapi-security to disable the scope verification.

```
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: false

```

Now, let's start api_d and test it out. We assume that the Jaeger All-in-One are up and running in the first step. 

```
cd ~/networknt/light-example-4j/discovery/api_d/open-tracing
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

Now, let's test the API D server. We can find the port on the terminal that starts the server or we can find the port on Consul UI. As we are using dynamic port, the first port allocated is 2400. 

```
curl -k https://localhost:7444/v1/data \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA' \
  -H 'Content-Type: application/json'
```

You should have something returned like

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444"]
```

Open the Jaeger UI with http://localhost:16686

You should see one more service called com.networknt.apid-1.0.0 in the Service dropdown. 

### API C

API C is the same like API D, it is called by API A. 

First we need to update pom.xml to add the dependency of jaeger-tracing. 

pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

In the handler.yml file, comment out the traceability and correlation and add the jaeger. We put the JaegerHandler after the specification as it depends on it to use the OpenAPI path for the tracer operation.


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
  - com.networknt.metrics.MetricsHandler@metrics
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  # Open Tracing Jaeger Handler
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.ac.handler.DataGetHandler


chains:
  default:
    - exception
    - metrics
    # - traceability
    # - correlation
    - specification
    - jaeger
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/v1/data'
    method: 'GET'
    exec:
      - default
      - com.networknt.ac.handler.DataGetHandler

  - path: '/health/com.networknt.ac-1.0.0'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```

Also, we need to update the service.yml to add JaegerStartupHookProvider. 

```
  - com.networknt.server.StartupHookProvider:
      - com.networknt.jaeger.tracing.JaegerStartupHookProvider
```

Finally, we need to add jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

To make sure that we can use the long-lived token to access API D, we need to change the openapi-security to disable the scope verification.

```
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: false

```


### API B

First we need to update pom.xml to add the dependency of jaeger-tracing. 

pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

In the handler.yml file, comment out the traceability and correlation and add the jaeger. We put the JaegerHandler after the specification as it depends on it to use the OpenAPI path for the tracer operation.

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
  - com.networknt.metrics.MetricsHandler@metrics
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  # Open Tracing Jaeger Handler
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.ab.handler.DataGetHandler


chains:
  default:
    - exception
    - metrics
    # - traceability
    # - correlation
    - specification
    - jaeger
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/v1/data'
    method: 'GET'
    exec:
      - default
      - com.networknt.ab.handler.DataGetHandler

  - path: '/health/com.networknt.ab-1.0.0'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```

Also, we need to update the service.yml to add JaegerStartupHookProvider. 

```
  - com.networknt.server.StartupHookProvider:
      - com.networknt.jaeger.tracing.JaegerStartupHookProvider
```

Finally, we need to add jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

To make sure that we can use the long-lived token to access API D, we need to change the openapi-security to disable the scope verification.

```
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: false

```

For API B, as it is calling API D, we need to update the client.yml to add the `injectOpenTracing: true`

```
# circuit breaker configuration for the client
request:
  # number of timeouts/errors to break the circuit
  errorThreshold: 2
  # timeout in millisecond to indicate a client error.
  timeout: 3000
  # reset the circuit after this timeout in millisecond
  resetTimeout: 7000
  # if open tracing is enable. traceability, correlation and metrics should not be in the chain if opentracing is used.
  injectOpenTracing: true

```


### API A

First we need to update pom.xml to add the dependency of jaeger-tracing. 

pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

In the handler.yml file, comment out the traceability and correlation and add the jaeger. We put the JaegerHandler after the specification as it depends on it to use the OpenAPI path for the tracer operation.

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
  - com.networknt.metrics.MetricsHandler@metrics
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  # Open Tracing Jaeger Handler
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.aa.handler.DataGetHandler


chains:
  default:
    - exception
    - metrics
    # - traceability
    # - correlation
    - specification
    - jaeger
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/v1/data'
    method: 'GET'
    exec:
      - default
      - com.networknt.aa.handler.DataGetHandler

  - path: '/health/com.networknt.aa-1.0.0'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```

Also, we need to update the service.yml to add JaegerStartupHookProvider. 

```
  - com.networknt.server.StartupHookProvider:
      - com.networknt.jaeger.tracing.JaegerStartupHookProvider
```

Finally, we need to add jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

To make sure that we can use the long-lived token to access API D, we need to change the openapi-security to disable the scope verification.

```
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: false

```

For API A, as it is calling other APIs, we need to update the client.yml to add the `injectOpenTracing: true`

```
# circuit breaker configuration for the client
request:
  # number of timeouts/errors to break the circuit
  errorThreshold: 2
  # timeout in millisecond to indicate a client error.
  timeout: 3000
  # reset the circuit after this timeout in millisecond
  resetTimeout: 7000
  # if open tracing is enable. traceability, correlation and metrics should not be in the chain if opentracing is used.
  injectOpenTracing: true

```


### Start Servers

API A 

```
cd ~/networknt/light-example-4j/discovery/api_a/open-tracing
mvn clean install -Prelease
java -jar target/aa-1.0.0.jar

```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/open-tracing
mvn clean install -Prelease
java -jar target/ab-1.0.0.jar

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/open-tracing
mvn clean install -Prelease
java -jar target/ac-1.0.0.jar

```

API D

```
cd ~/networknt/light-example-4j/discovery/api_d/open-tracing
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar

```

### Test

Issue the command below. 


```
curl -k https://localhost:7441/v1/data \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA' \
  -H 'Content-Type: application/json'

```

And the normal result should look like this. 

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Let's go to the Jaeger UI at http://localhost:16686/ and search Service com.networknt.aa-1.0.0 and Operation /data@get


![jaeger-discovery](/images/jaeger-discovery.png)

For the above picture, you can clearly see the call tree and time spent on each service. If you have database call or other important task in each service, you can manually intrument them and you will have a clear picture how each service works and the performance of each task. This give us valueable information if something happens on production. For more details on how to intrument light-4j service, please review the [petstore][] tracing tutorial. 


[petstore]: /tutorial/tracing/jaeger/petstore-jaeger/
[service discovery]: /tutorial/common/discovery/
