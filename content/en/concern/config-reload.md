---
title: "Config Reload"
date: 2022-11-11T17:49:30-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When a light-4j application is running in a Kubernetes cluster as a microservice or http-sidecar, it is easy to restart the server to pick up the latest configuration from the config server or local filesystem. However, when running light-gateway as a shared service for multiple consumers and providers, it is impossible to restart the server without impacting others, when onboarding a new consumer or a new provider, several middleware handlers must reload the configuration changes without shutting down the server.

All middleware handlers have a reload method to refresh the configuration from the source. There are two admin endpoints to get all the modules and trigger reload for a list of modules. 

### Specification

The specification is injected to any light-rest-4j application via [openapi-inject.yml](https://github.com/networknt/light-rest-4j/blob/master/openapi-meta/src/main/resources/config/openapi-inject.yml) when the application specific openapi.yaml file is loaded. 

Here are the endpoint definitions.

```
  /adm/modules:
    get:
      description: to get the all registered modules and handlers to form a dropdown for module config reload
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
    post:
      description: to trigger the config reload for all modules or specified modules selected on control pane
      requestBody:
        description: to trigger the reload from the config server from the control pane.
        required: true
        content:
          application/json:
            schema:
              type: array
              items:
                type: string
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
```

The first endpoint /adm/modules@get will return all the registered modules from the registry. Note that some modules in the list do not support config reload. 

The second endpoint /adm/modules@post will send a list of modules to the server to force a config-reload for them. 

### Configuration

If you use light-codegen to generate a light-rest-4j application, the handler.yml will have these endpoints configured. Also, you can find the examples in http-sidecar and light-gateway repositories. 

handler.yml

```
  .
  .
  .
  - path: '/adm/modules'
    method: 'get'
    exec:
      - admin
      - modules

  - path: '/adm/modules'
    method: 'post'
    exec:
      - admin
      - configReload
  .
  .
  .
```

If the handler.chains.default is externalized, you need to register the handler classes and give aliases to them in the values.yml file.

values.yml

```
handler.handlers:
  .
  .
  .
  - com.networknt.config.reload.handler.ModuleRegistryGetHandler@modules
  - com.networknt.config.reload.handler.ConfigReloadHandler@configReload
```

The config-reload module has its configuration to control if the module is enabled or not. 

Here is the default configreload.yml

```
---
# config reload from config server.

# Indicate if the config reload from config server  is enable or not.
enabled: ${configreload.enabled:true}
```

You can disable the config-reload module in the values.yml file to overwrite the default value. 

### Security

Depending on how the security is configured on the server, you need to get a token and put it into the Authorization header to access the config-reload endpoints. If the light portal is used, the request can be sent from the control pane to each individual service instance. When a user logs in to the light portal, a token will be created with the proper scope to access the admin endpoints. 

You can disable the security in the values.yml file for a simple local test. 

values.yml

```
# security.yml
security.enableVerifyJwt: false
```

### Dependency

Add dependency in the application pom.xml file.

```xml
<dependency>
  <groupId>com.networknt</groupId>
  <artifactId>config-reload</artifactId>
  <version>${version.light-4j}</version>
</dependency>
```

### Usage

To get all registered modules, issue the following command. 

```
curl -k --location --request GET 'https://localhost:8443/adm/modules'
```

And here is the response.

```
[
    "com.networknt.server.Server",
    "com.networknt.mask.Mask",
    "com.networknt.limit.LimitHandler",
    "com.networknt.handler.Handler",
    "com.networknt.handler.ResponseInjectionConfig",
    "com.networknt.chaos.MemoryAssaultHandler",
    "com.networknt.exception.ExceptionHandler",
    "com.networknt.handler.ProxyHandler",
    "com.networknt.openapi.ValidatorHandler",
    "com.networknt.body.BodyHandler",
    "com.networknt.chaos.LatencyAssaultHandler",
    "com.networknt.router.middleware.PathPrefixServiceHandler",
    "com.networknt.header.HeaderHandler",
    "com.networknt.proxy.ExternalServiceConfig",
    "com.networknt.correlation.CorrelationHandler",
    "com.networknt.router.RouterHandler",
    "com.networknt.audit.AuditHandler",
    "com.networknt.chaos.KillappAssaultHandler",
    "com.networknt.traceability.TraceabilityHandler",
    "com.networknt.service.SingletonServiceFactory",
    "com.networknt.handler.RequestInjectionConfig",
    "com.networknt.cors.CorsHttpHandler",
    "com.networknt.openapi.JwtVerifyHandler",
    "com.networknt.chaos.ExceptionAssaultHandler",
    "com.networknt.status.Status",
    "com.networknt.router.GatewayRouterHandler",
    "com.networknt.client.Http2Client",
    "com.networknt.openapi.OpenApiHandler"
]
```

To reload the configuration for LimitHandler and PathPrefixServiceHandler. 

```
curl -k --location --request POST 'https://localhost:8443/adm/modules' \
--header 'Content-Type: application/json' \
--data-raw '[
    "com.networknt.limit.LimitHandler",
    "com.networknt.router.middleware.PathPrefixServiceHandler"
]
'
```

And the response will list all the modules have been reloaded. 

```
[
    "com.networknt.limit.LimitHandler",
    "com.networknt.router.middleware.PathPrefixServiceHandler"
]
```



