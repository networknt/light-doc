---
title: "Proxy Configuration"
date: 2017-12-16T09:35:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The light-proxy is a RESTful proxy server that provides a lot of features other
proxy servers cannot provide. This document will explain all user cases and map
to certain configurations to deploy the proxy server based on the requirements.

##  proxy.yml

Here is an example of proxy.yml

```yaml
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: true

# If TLS is enabled when connecting to the target servers
httpsEnabled: true

# Target URIs
hosts: https://localhost:8081,https://localhost:8082,https://localhost:8083

# Connections per thread to the target servers
connectionsPerThread: 20

# Max request time in milliseconds before timeout
maxRequestTime: 10000

```

### Load Balance

To enable load balance to multiple backend service instances, you need to config
proxy.yml with hosts property that points to multiple instances with comma dividing
them. 


In the above example, you can see that there are three instances of backend service 
on the same host. In the official deployment, there might be three services listening 
to the same port on three different hosts. 

Although there are three instances in the config, if one or two instances are down
for some reasons, the proxy server knows how to fail over the request to the live
instance(s). It only give caller an error if all instances are down. 


### Http 2.0 Connection to backend services

Most of our customers are using light-proxy to wrap the existing API to provide security
, metrics or other cross-cutting concerns and these backend services are normally won't
support HTTP 2.0 as it is pretty new and most servers on the market won't support it. In
this case, you need to set http2Enabled to false. 

However, there is one use case that uses light-proxy to aggregate services built on top
of light-rest-4j and provide webserver to host Single Page Application(Angular, React etc.).
You should set http2Enabled to true as this will speed up the communication between proxy
server and backend services. 

For the inbound request, the proxy server will always support HTTP 2.0 but can downgrade
to HTTP 1.1 if necessary. 

### Https Connection to backend services

If your light-proxy instance and your backend service instances are not on the same
host, it is recommended to use TLS connection between proxy server and backend servers. 

To enable it, just change the httpsEnabled to true and create client.keystore and
client.truststore with client keys and certificates. 

For information on how to create client.keystore and client.truststore, please refer to
[keystore truststore][] 

### Connection Pooling to backend services

For HTTP 1.1 connection to backend services, you must setup connection pool to backend
services unless you only have in-frequent requests to your proxy. If there might be
concurrent requests, then set connectionsPerThread to a proper number. The number 20 in
above example should work in most of the cases. 

If the connection to backend is in HTTP 2.0, the connection pool is only optional if
you have huge volume to pass through, for example, over 100K request per second. In
most cases, it is not necessary but it is good to have a connection to buffer the load.

### Request timeout to backend services

This is the timeout period if backend instance is too slow to respond. In above example,
it is set as 10 seconds. You should set it properly based on the normal response time
of you backend service. In general, you should set this number as small as possible.

## security.yml

You need to externalize security.yml in order to enable JWT verification. Please be aware
that [light-oauth2](https://github.com/networknt/light-oauth2) must be available or other
OAuth 2.0 provider must be available. 

Regarding to the details of security.yml, please see [security][] in light-4j. 

## metrics.yml

For client owners, they want to know how many services they are call and how many succeeded
and how many failed. Also they need to know the response time per endpoint. 

From service owners' perspective, they need to know how many clients are calling their services
and how many failed or succeeded and SLA statistic. The information can be used to charge the
client on the usage if the service is paid service. 

Light-4j provide a middleware handler called MetricsHandle that will collect the metrics info
the influxDB and allow users to view the dashboard from Grafana web interface. 

Here is the default metrics.yml in metrics module.

```yaml
# Metrics handler configuration

# If metrics handler is enabled or not
enabled: false

# influxdb protocal can be http, https
influxdbProtocol: http
# influxdb hostname
influxdbHost: localhost
# influxdb port number
influxdbPort: 8086
# influxdb database name
influxdbName: metrics
# influxdb user
influxdbUser: admin
# influx db password
influxdbPass: admin
# report and reset metrics in minutes.
reportInMinutes: 1
``` 

To enable it you need to externalize this file and change enabled to true. Also, you need
to update the protocol, host, port, user and password for influxdb. 

For more details regarding to the configuration for MetricsHandler, please refer to [metrics][]


## server.yml

The light-proxy is a server and it contains everything a normal API server has. A server.yml
should be externalized to control how the server is started. 

Here is an example of server.yml

```yaml
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort: 8080

# Enable HTTP should be false by default.
enableHttp: false

# Https port if enableHttps is true.
httpsPort: 8443

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: tls/server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: tls/server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.petstore-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1
```

For more information about this config file please refer to [server][]

## secret.yml

For every server, it might call another server as a client. In secret.yml file, all server
and client secrets are defined here. This file is treated special in term of visibility and
it will map to the Security in Kubernetes if it is used. 

Here is an example of secret.yml

```yaml
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
# consulToken: the_one_ring

```

For more info about this config file, please refer to [secret][] section here.

## swagger.json

One of the reasons to use light-proxy is to enable security and request validation provided
by light-rest-4j framework. In order to do that, swagger.json must be provided as a config
file for the framework. This file should be copied from the backend service or constructed
from backend service code if there is none exists. For more information on proxy with Swagger
specification, please refer to [Swagger proxy][].


## openapi.json

Light-proxy support Swagger 2.0 and OpenAPI 3.0 specification and if your backend service has
OpenAPI 3.0 specification, you can use it too in light-proxy. Also you need to replace all the
handlers that handle swagger.json with handlers for openapi.json. For more details, please refer
to [OpenAPI proxy][].

## service.yml

This config file controls how each component wired into the application with an IoC service
map loaded during startup. 

For swagger specification, it should be something look like this.

```yaml
# Singleton service factory configuration/IoC injection
singletons:
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
  - com.networknt.proxy.ProxyHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Swagger Parsing swagger specification based on request uri and method.
  - com.networknt.swagger.SwaggerHandler
  # Security JWT token verification and scope verification (depending on SwaggerHandler)
  - com.networknt.security.JwtVerifyHandler
  # Body Parse body based on content type in the header. It must be disabled otherwise the body will be consumed by proxy.
  # - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on swagger specification (depending on Swagger and Body)
  # As BodyHandler is disabled above, this must be disabled until the issue with keep stream is fixed.
  # - com.networknt.validator.ValidatorHandler
  # Header handler to manipulate request and/or response headers before or after downstream server
  - com.networknt.header.HeaderHandler

```

If you are using OpenAPI 3.0 specification, then the service.yml should be looked like this.

```yaml
# Singleton service factory configuration/IoC injection
singletons:
# OpenAPI parser validator implementation
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Callback>:
  - com.networknt.oas.validator.impl.CallbackValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Contact>:
  - com.networknt.oas.validator.impl.ContactValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.EncodingProperty>:
  - com.networknt.oas.validator.impl.EncodingPropertyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Example>:
  - com.networknt.oas.validator.impl.ExampleValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ExternalDocs>:
  - com.networknt.oas.validator.impl.ExternalDocsValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Header>:
  - com.networknt.oas.validator.impl.HeaderValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.License>:
  - com.networknt.oas.validator.impl.LicenseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Info>:
  - com.networknt.oas.validator.impl.InfoValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Link>:
  - com.networknt.oas.validator.impl.LinkValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Schema>:
  - com.networknt.oas.validator.impl.SchemaValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.MediaType>:
  - com.networknt.oas.validator.impl.MediaTypeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OAuthFlow>:
  - com.networknt.oas.validator.impl.OAuthFlowValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OpenApi3>:
  - com.networknt.oas.validator.impl.OpenApi3Validator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.RequestBody>:
  - com.networknt.oas.validator.impl.RequestBodyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Operation>:
  - com.networknt.oas.validator.impl.OperationValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Parameter>:
  - com.networknt.oas.validator.impl.ParameterValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Path>:
  - com.networknt.oas.validator.impl.PathValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Response>:
  - com.networknt.oas.validator.impl.ResponseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityRequirement>:
  - com.networknt.oas.validator.impl.SecurityRequirementValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityScheme>:
  - com.networknt.oas.validator.impl.SecuritySchemeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Server>:
  - com.networknt.oas.validator.impl.ServerValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ServerVariable>:
  - com.networknt.oas.validator.impl.ServerVariableValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Tag>:
  - com.networknt.oas.validator.impl.TagValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Xml>:
  - com.networknt.oas.validator.impl.XmlValidator
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
  - com.networknt.proxy.ProxyHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Parsing OpenAPI 3.0 specification based on request uri and method.
  - com.networknt.openapi.OpenApiHandler
  # Security JWT token verification and scope verification (depending on OpenApiHandler)
  - com.networknt.openapi.JwtVerifyHandler
  # Body Parse body based on content type in the header. It must be disabled otherwise the body will be consumed by proxy.
  # - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on swagger specification (depending on Swagger and Body)
  # - com.networknt.validator.ValidatorHandler
  # Header handler to manipulate request and/or response headers before or after downstream server
  - com.networknt.header.HeaderHandler

```


## validator.yml

Please note that validator is disabled in the current release due to BodyHandler cannot
keep the stream at the moment. There is an [issue][] opened to address this in the future.

This is the config file that enables request validation based on swagger.json or openapi.json 
file which is injected into the config folder. 

Here is an example.

```yaml
# Enable request validation. Response validation is not done on the server but client.
enabled: true
``` 

For more details about this component, please refer to [swagger-validator][]
or [openapi-validator][].

## sanitizer.yml

This config file controls how sanitizer work in the framework. It is enabled by default
and you don't need to externalize the file normally if the default behaviour is desired.

Here is an example of sanitizer.yml

```yaml
# Sanitize request for cross site scripting during runtime

# indicate if sanitizer is enabled or not
enabled: true

# if it is enabled, does body need to be sanitized
sanitizeBody: true

# if it is enabled, does header need to be sanitized
sanitizeHeader: false
```

For more information about this component, please refer to [sanitizer][]

## Other default handlers

There are some other default handlers that are plugged into the light-proxy. And normally, 
you shouldn't need to touch these config files unless you know what you are doing. 

* audit.yml
* correlation.yml
* traceability.yml
* cors.yml if it is used
* limit.yml if it is used

[keystore truststore]: /tutorial/security/keystore-truststore/
[security]: /concern/security/
[metrics]: /concern/metrics/
[server]: /concern/server/
[secret]: /tutorial/security/encrypt-decrypt/
[OpenAPI proxy]: /tutorial/proxy/openapi-proxy/
[Swagger proxy]: /tutorial/proxy/swagger-proxy/
[issue]: https://github.com/networknt/light-proxy/issues/1
[swagger-validator]: /style/light-rest-4j/swagger-validator/
[openapi-validator]: /style/light-rest-4j/openapi-validator/
[sanitizer]: /concern/sanitizer/
