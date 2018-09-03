---
title: "Openapi Proxy"
date: 2017-12-16T10:43:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In this second par of the tutorial, we are going to use OpenAPI 3.0 RESTful 
API as an example to show how to deployed the light-proxy and enable all sorts 
of middleware handlers through configurations. We also explore standalone deployment 
and Docker deployment. 

With specification handler, security handler and validation handler change, you
can easily update the configuration to make the same instance of light-proxy
server work with OpenAPI 3.0 or GraphQL backend. The default configuration is for 
RESTful Swagger 2.0 or OpenAPI 3.0 backend as light-rest-4j middleware handlers are 
included in the package.

To make it simple, we will build a backend service with light-rest-4j with a
OpenAPI 3.0 specification file defined in [model-config][] 
repo. In real production environment, the backend service can be built with other 
Java framework or other languages like Nodejs, Go etc.

If you have an existing API backend up and running already, then you can continue.
Otherwise, follow this [backend tutorial][]
to create a backend service and start multiple instances.

## light-proxy configuration

With the backend services ready, let's clone the light-proxy and make some configuration
changes. 

```
cd ~/networknt
git clone https://github.com/networknt/light-proxy.git
```

By default, the light-proxy doesn't have any config files packaged into the src/main/resources/config
folder as production deployment will have all the config files externalized. Now, let's
copy the config files from src/test/resources/config and make some modifications.

```
cd ~/networknt/light-proxy/src/main/resources
mkdir config
cp -r ../../test/resources/config/* config 
```

Now you can add a proxy.yml in light-proxy/src/main/resources/config folder as following.

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

As we are going to enable Security and Validator based on swagger.json from backend server,
let's copy the swagger.json from light-example-4j/rest/swagger/proxy-backend/src/main/resources/config folder.

```
cd ~/networknt
cp light-example-4j/rest/openapi/proxy-backend/src/main/resources/config/openapi.json light-proxy/src/main/resources/config/
```

## Update service.yml

Now we have copied the openapi.json into the config folder of the proxy. The next step, we need to
update service.yml to replace all the Swagger 2.0 handlers with OpenAPI 3.0 handlers in middleware
chain. 

Here is the default service.yml that is copied from test folder to main folder.

```yaml
# Singleton service factory configuration/IoC injection
singletons:
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
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

Here is the updated version with com.networknt.swagger.SwaggerHandler and com.networknt.security.JwtVerifyHandler
replaced. Also, we need add the section to that inject openapi-parser validators. 

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
- com.networknt.handler.HandlerProvider:
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

## start light-proxy and test

```
cd ~/networknt/light-proxy
mvn clean install -DskipTests
mvn exec:exec
```

Given the light-proxy server.yml only have https enabled and listening to port 8080, let's
use curl to test it.

```
curl -k https://localhost:8080/v1/getData
```

And we get result:

```
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8082,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8083,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8081,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8082,"key":"key1"}
```

As you can see the proxy server is working on https connection to the backend services. And 
the response is from different instances for each request due to load balance. You can stop 
one of the three instances and the proxy will automatically failover to working instances.

Also, you can test the post request with the following command line.

```
curl -k -X POST \
  https://localhost:8080/v1/postData \
  -H 'content-type: application/json' \
  -d '{"key":"key1", "value": "value1"}'
```

Result: 

```json
{"enableHttps":true,"value":"value1","httpsPort":8083,"key":"key1","enableHttp2":true,"httpPort":8080}
```

Above steps shows how to start the backend services and to config the light-proxy so that it
can pass through the requests to the backend instances in a round robin fashion. 

For light-proxy configuration, we just updated it in the app config folder to add a proxy.yml 
and copy over the swagger.json from proxy-backend service. This will enable the proxy 
server to do security verification (disabled by default) based on JWT token and scopes. 

The reason we do in app configuration is to show you how this has been done within the application
context. In the normal way, you should have light-proxy jar file or Docker image as deliver
package and all configuration should be done externally. 

For information on how to pass the configuration files into a standalone service please refer
to this [config module][]

For information on how to pass the configuration files into a Docker container please refer
to this [config module][]

For information on how to enable features of light-proxy with config files, please refer to
light-proxy [configuration][] 

[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi/proxy-backend
[backend tutorial]: /tutorial/proxy/openapi-backend/
[config module]: /concern/config/
[configuration]: /service/proxy/configuration/
