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
reviewed: true
---

In this second part of the tutorial, we are going to use OpenAPI 3.0 RESTful API as an example to show how to deploy the light-proxy and enable all sorts of middleware handlers through configurations. We also explore standalone deployment and Docker deployment. 

With the specification handler, security handler, and validation handler change, you can easily update the configuration to make the same instance of light-proxy server work with OpenAPI 3.0 or GraphQL backend. The default configuration is for RESTful Swagger 2.0 or OpenAPI 3.0 backend as light-rest-4j middleware handlers are included in the package.

To make it simple, we will build backend service with light-rest-4j with an OpenAPI 3.0 specification file defined in [model-config][] repo. In a real production environment, the backend service can be built with other Java frameworks or other languages like Nodejs, Go, etc.

If you have an existing API backend up and running already, then you can continue. Otherwise, follow this [backend tutorial][] to create a backend service and start multiple instances.

## light-proxy configuration

With the backend services ready, let's clone the light-proxy and make some configuration changes. 

```
cd ~/networknt
git clone https://github.com/networknt/light-proxy.git
```

By default, the light-proxy doesn't have any config files packaged into the src/main/resources/config folder, as production deployment will have all the config files externalized. This is required by some of the enterprise users due to security concerns. Now, let's put the config files into an externalized folder at https://github.com/networknt/light-config-test/tree/master/light-proxy/openapi-proxy

The first config file that we need is a proxy.yml.

```yaml
---
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
maxRequestTime: 1000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3
``` 

As we are going to enable Security and Validator based on openapi.yaml from the backend server, let's copy the openapi.yaml from light-example-4j/rest/openapi/proxy-backend/src/main/resources/config folder.

```
cd ~/networknt
cp light-example-4j/rest/openapi/proxy-backend/src/main/resources/config/openapi.yaml light-config-test/light-proxy/openapi-proxy/
```

Next, let's create a handler.yml file to control the middleware handler chain to wire in some of the cross-cutting concerns. 

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
  - path: '/getData'
    method: 'GET'
    exec:
      - default
  - path: '/postData'
    method: 'GET'
    exec:
      - default
  - path: '/health/com.networknt.backend-1.0.0'
    method: 'get'
    exec:
      - health

  # In most case, the /server/info endpoint shouldn't be exposed. If it is, then it must be protected by OAuth 2.0 or Basic Auth
  - path: '/server/info'
    method: 'get'
    exec:
      - info

```

Please note that the serviceId in the health endpoint is using the backend serviceId to allow consumers to treat the proxy instance as the backend service. This is a common practice. 

We also need to copy the server.yml, server.keystore, client.yml and client.truststore from the server and client module from light-4j. 

```
cd ~/networknt
cp light-4j/server/src/main/resources/config/server.keystore light-config-test/light-proxy/openapi-proxy
cp light-4j/server/src/main/resources/config/server.yml light-config-test/light-proxy/openapi-proxy
cp light-4j/client/src/main/resources/config/client.truststore light-config-test/light-proxy/openapi-proxy
cp light-4j/client/src/main/resources/config/client.yml light-config-test/light-proxy/openapi-proxy
```

Update the server.yml to change the httpsPort to 8080 and serviceId to com.networknt.backend-1.0.0. 

The following is the server.yml after updating. 

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort: 8080

# Enable HTTP should be false by default.
enableHttp: false

# Https port if enableHttps is true.
httpsPort: 8080

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Keystore password
keystorePass: password

# Private key password
keyPass: password

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Truststore password
truststorePass: password

# Bootstrap truststore name used to connect to the light-config-server if it is used.
bootstrapStoreName: bootstrap.truststore

# Bootstrap truststore password
bootstrapStorePass: password

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.backend-1.0.0

# Unique service name. Used in microservices to associate a given name to a service with configuration 
# or as a key within the configuration of a particular domain
serviceName: petstore

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

# Build number, to be set by teams for auditing purposes. 
# This build number is independent of the API version, and it set by teams according to their release management process
buildNumber: latest

```

As we are using localhost with the self-signed certificate, we need to set the verifyHostname in the client.yml to false. Here is the client.yml after updating.

```
# This is the configuration file for Http2Client.
---
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # The default trustedNames group used to created default SSL context. This is used to create Http2Client.SSL if set.
  defaultGroupKey: trustedNames.local
  # trusted hostnames, service names, service Ids, and so on.
  # Note: localhost and 127.0.0.1 are not trustable hostname/ip in general. So, these values should not be used as trusted names in production.
  trustedNames: 
    local: localhost
    negativeTest: invalidhost 
    empty:
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: client.truststore
  # trust store password
  trustStorePass: password
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: client.keystore
  # key store password
  keyStorePass: password
  # private key password
  keyPass: password
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
    cache:
      #capacity of caching TOKENs
      capacity: 200
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow.
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # client_secret for authorization code grant flow.
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://localhost:8080/authorization_code
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # client_secret for client credentials grant flow.
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    refresh_token:
      # token endpoint for refresh token grant
      uri: "/oauth2/token"
      # client_id for refresh token grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # client_secret for refresh token
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    # light-oauth2 key distribution endpoint configuration for token verification
    key:
      # key distribution server url for token verification. It will be used if it is configured.
      server_url: https://localhost:6886
      # key serviceId for key distribution service, it will be used if above server_url is not configured.
      serviceId: com.networknt.oauth2-key-1.0.0
      # the path for the key distribution endpoint
      uri: "/oauth2/key"
      # client_id used to access key distribution service. It can be the same client_id with token service or not.
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # client secret used to access the key distribution service.
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: true
  # sign endpoint configuration
  sign:
    # token server url. The default port number for token service is 6882. If this url exists, it will be used.
    server_url: https://localhost:6882
    # token serviceId. If server_url doesn't exist, the serviceId will be used to lookup the token service.
    serviceId: com.networknt.oauth2-token-1.0.0
    # signing endpoint for the sign request
    uri: "/oauth2/token"
    # timeout in milliseconds
    timeout: 2000
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # client_id for client authentication
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client secret for client authentication and it can be encrypted here.
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
    # the key distribution sever config for sign. It can be different then token key distribution server.
    key:
      # key distribution server url. It will be used to establish connection if it exists.
      server_url: https://localhost:6886
      # the unique service id for key distribution service, it will be used to lookup key service if above url doesn't exist.
      serviceId: com.networknt.oauth2-key-1.0.0
      # the path for the key distribution endpoint
      uri: "/oauth2/key"
      # client_id used to access key distribution service. It can be the same client_id with token service or not.
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # client secret used to access the key distribution service.
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: true
  # de-ref by reference token to JWT token. It is separate service as it might be the external OAuth 2.0 provider.
  deref:
    # Token service server url, this might be different than the above token server url. The static url will be used if it is configured.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider. Need for service lookup/discovery. It will be used if above server_url is not configured.
    serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # the path for the key distribution endpoint
    uri: "/oauth2/deref"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client_secret for deref
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
# circuit breaker configuration for the client
request:
  # number of timeouts/errors to break the circuit
  errorThreshold: 2
  # timeout in millisecond to indicate a client error.
  timeout: 3000
  # reset the circuit after this timeout in millisecond
  resetTimeout: 7000
  # if open tracing is enable. traceability, correlation and metrics should not be in the chain if opentracing is used.
  injectOpenTracing: false
  # the flag to indicate whether http/2 is enabled when calling client.callService()
  enableHttp2: true
  # the maximum host capacity of connection pool
  connectionPoolSize: 1000
  # the maximum request limitation for each connection
  maxReqPerConn: 1000000
  # maximum quantity of connection in connection pool for each host
  maxConnectionNumPerHost: 1000
  # minimum quantity of connection in connection pool for each host. The corresponding connection number will shrink to minConnectionNumPerHost
  # by remove least recently used connections when the connection number of a host reach 0.75 * maxConnectionNumPerHost.
  minConnectionNumPerHost: 250
```



## start light-proxy and test

```
cd ~/networknt/light-proxy
mvn clean install -DskipTests
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-proxy/openapi-proxy -jar target/light-proxy.jar
```

Please note that the configuration folder might be different on your computer. 

Given the light-proxy server.yml only have https enabled and listening to port 8080, let's use curl to test it.

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

As you can see, the proxy server is working on an https connection to the backend services. And the response is from different instances for each request due to load balance. You can stop one of the three instances, and the proxy will automatically failover to working instances.

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

To check if the light-proxy instance can block the invalid request, issue the following command. 

```
curl -k -X POST \
  https://localhost:8080/v1/postData \
  -H 'content-type: application/json' \
  -d '{"key1":"key1", "value1": "value1"}'
```

You will receive the following response. 

```
{"statusCode":400,"code":"ERR11004","message":"VALIDATOR_SCHEMA","description":"Schema Validation Error - $.key: is missing but it is required","severity":"ERROR"}
```
If you turn on the debug on the light-proxy server, you can see the above error in the log. 


The above steps show how to start the backend services and config the light-proxy so that it can pass through the requests to the backend instances in a round-robin fashion. 

For light-proxy configuration, we just added minimum configuration files in the externalized config folder. The openapi.yaml from the proxy-backend was added to enable the proxy server to do security verification (disabled by default) based on JWT token and scopes. It also enables the proxy server to validate the request based on the OpenAPI specification. 

In most cases, you should have a light-proxy jar file or Docker image as a delivery package, and all configuration should be done externally. 

For information on how to pass the configuration files into a standalone service, please refer to this [config module][]

For information on how to pass the configuration files into a Docker container, please refer to this [config module][]

For information on how to enable features of light-proxy with config files, please refer to light-proxy [configuration][] 

A video walkthrough in English can be found [here](https://www.youtube.com/watch?v=NCZYLb7toow)

A video walkthrough in Chinese can be found [here](https://v.youku.com/v_show/id_XNDUwMjgwODQzMg==.html)


[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi/proxy-backend
[backend tutorial]: /tutorial/proxy/openapi-backend/
[config module]: /concern/config/
[configuration]: /service/proxy/configuration/
