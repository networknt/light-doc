---
title: "handler.yml"
date: 2019-01-09T10:24:06-05:00
description: ""
categories: [configs]
keywords: []
aliases: []
toc: false
draft: false
---
 handler.yml is the configuration file of handler middleware chain
#### handlers:
 list of handlers to be used across chains in this microservice including the routing handlers for ALL endpoints  
 format: fully qualified handler class name@optional:given name  
 below are default integrated handlers after code generation:  
  [- com.networknt.exception.ExceptionHandler@exception](/concern/exception)    
  [- com.networknt.metrics.MetricsHandler@metrics](/concern/metrics)  
  [- com.networknt.traceability.TraceabilityHandler@traceability](/concern/traceability)  
  [- com.networknt.correlation.CorrelationHandler@correlation](/concern/correlation)  
  [- com.networknt.openapi.OpenApiHandler@specification](/style/light-rest-4j/openapi-meta/)  
  [- com.networknt.openapi.JwtVerifyHandler@security](/concern/security)  
  [- com.networknt.body.BodyHandler@body](/concern/body)  
  [- com.networknt.audit.AuditHandler@audit](/concern/audit)  
  [- com.networknt.dump.DumpHandler@dump](/concern/dump)  
  [- com.networknt.sanitizer.SanitizerHandler@sanitizer](/concern/sanitizer)  
  [- com.networknt.openapi.ValidatorHandler@validator](/concern/validator)  
  [- com.networknt.health.HealthGetHandler@health](/concern/health)  
  [- com.networknt.info.ServerInfoGetHandler@info](/concern/info)  
  [- com.networknt.specification.SpecDisplayHandler@spec](/style/light-rest-4j/openapi-display)  
  [- com.networknt.specification.SpecSwaggerUIHandler@swaggerui](/style/light-rest-4j/openapi-display)  
  
#### chains:
To predefine handler chains so that can be reused in paths. Each handler in a chain will be executed by sequence.
Some handlers have dependencies of other handlers, to have a correct sequence is important.
default:  
    - exception  
    - metrics  
    - traceability  
    - correlation  
    - specification  
    - security  
    - body  
    - audit  
    - dump   
    - sanitizer  
    - validator
    
#### paths:
Defined paths will be intercepted based on "path" and "method" property. Then will execute handlers and handler chains
that defined in "exec" property

#### Config Example
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
  - com.networknt.traceability.TraceabilityHandler@traceability
  - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  - com.networknt.dump.DumpHandler@dump
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
  - com.networknt.schedule.handler.TodoItemsGetHandler
  - com.networknt.schedule.handler.TodoItemPostHandler
  - com.networknt.schedule.handler.TodoItemItemIdGetHandler
  - com.networknt.schedule.handler.TodoItemItemIdDeleteHandler
  - com.networknt.schedule.handler.TodoItemItemIdPutHandler


chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - specification
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/todoItems'
    method: 'GET'
    exec:
      - default
      - com.networknt.schedule.handler.TodoItemsGetHandler
  - path: '/todoItem'
    method: 'POST'
    exec:
      - default
      - com.networknt.schedule.handler.TodoItemPostHandler
  - path: '/todoItem/{itemId}'
    method: 'GET'
    exec:
      - default
      - com.networknt.schedule.handler.TodoItemItemIdGetHandler
  - path: '/todoItem/{itemId}'
    method: 'DELETE'
    exec:
      - default
      - com.networknt.schedule.handler.TodoItemItemIdDeleteHandler
  - path: '/todoItem/{itemId}'
    method: 'PUT'
    exec:
      - default
      - com.networknt.schedule.handler.TodoItemItemIdPutHandler

  - path: '/health/com.networknt.schedule-0.0.1'
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