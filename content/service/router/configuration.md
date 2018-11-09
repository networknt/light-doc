---
title: "Router Configuration"
date: 2018-02-12T14:33:24-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


The light-router is a client routing server that provides a lot of features other
proxy servers cannot provide. This document will explain all user cases and map
to certain configurations to deploy the light-router server based on the requirements.

##  router.yml

Here is an example of router.yml

```yaml
# Light Router Configuration
# As this router is built to support discovery and security for light-4j services,
# the outbound connection is always HTTP 2.0 with TLS enabled.

# If HTTP 2.0 protocol will be accepted from incoming request.
http2Enabled: false

# If TLS is enabled when accepting incoming request. Should be true on test and prod.
httpsEnabled: false

# Max request time in milliseconds before timeout to the server.
maxRequestTime: 1000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3
```

### Http 2.0 Connection from client

Most of our customers are using light-router for legacy clients or client that is built
with other languages instead of Java. Chances are these clients are not support HTTP 2.0
neither. If this is the case, you need to setup http2Enabled as false. 

If you don't know if your client can send HTTP 2.0 request, then you can enable it and
the light-router can downgrade itself to HTTP 1.1 if the client doesn't support HTTP 2.0

For all outbound traffic, HTTP 2.0 will be used by default as the assumption is that the
services are built on top of light platform. 

### Https Connection to from client

If your light-router instance and your client instance are not on the same host, it is 
recommended to use TLS connection between router server and you client.  

To enable it, just change the httpsEnabled to true and create client.truststore and
server.keystore with server keys and certificates. 

For information on how to create server.keystore, server.truststore, client.keystore and 
client.truststore, please refer to [keystore truststore][] 

The outbound connection to light-4j services will always use TSL on official test env
and production. 

### Request timeout to backend services

This is the timeout period if backend instance is too slow to respond. In above example,
it is set as 1 seconds. You should set it properly based on the normal response time
of you backend service. In general, you should set this number as small as possible.

If you backend service is slow, you might see this error message in the light-router log. 

```
ERROR io.undertow.proxy cancel - UT005027: Timing out request to ...
```

When you see this error message, you need to bump up the timeout in the configuraiton and restart the light-router. 

## client.yml

If you are thinking from service perspective, the light-router is an client for them. So
there must be a client.yml defined in light-router. Here is an example. 

```yaml
# This is the configuration file for Http2Client.
---
# Buffer Size in the buffer pool in KB. If should be bigger than your request or response body size.
bufferSize: 24
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: client.keystore
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
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
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
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
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    refresh_token:
      # token endpoint for refresh token grant
      uri: "/oauth2/token"
      # client_id for refresh token grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
  # light-oauth2 key distribution endpoint configuration
  key:
    # key distribution server url
    server_url: https://localhost:6886
    # the unique service id for key distribution service
    serviceId: com.networknt.oauth2-key-1.0.0
    # the path for the key distribution endpoint
    uri: "/oauth2/key"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
  # de-ref by reference token to JWT token. It is separate service as it might be the external OAuth 2.0 provider.
  deref:
    # Token service server url, this might be different than the above token server url.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider. Need for service lookup/discovery.
    serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # the path for the key distribution endpoint
    uri: "/oauth2/deref"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
```

As you have seen that the interaction with OAuth 2.0 provider is defined in client.yml and the light-router will be responsible for retrieving JWT token and renewing JWT token. 

Also, the client.truststore is defined and loaded to support outbound TLS connection. If you want to support Two-Way TLS, then you need to loadKeyStore to true and put the client key into client.keystore.

If you have request body bigger than 24*1024, then you need to adjust bufferSize in the client.yml file. For more details, please refer to https://github.com/networknt/light-example-4j/tree/master/router
  
## server.yml

The light-router is a server and it contains everything a normal API server has. A server.yml
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

## service.yml

This config file controls how each component wired into the application with an IoC service
map loaded during startup. 

It should be something look like this.

```yaml
# Singleton service factory configuration/IoC injection
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.test-1.0.0: https://localhost:8081,https://localhost:8082,https://localhost:8083
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.router.RouterHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception middleware that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Header middleware to manipulate request and/or response headers before or after downstream server
  - com.networknt.header.HeaderHandler

```

## token.yml

This is the config file for TokenHandler which is a middleware handler for getting JWT token 
from OAuth 2.0 provider. Here is an example. 

```yaml
enabled: true
```

## pathService.yml

This is the config file for [PathServiceHandler][] which is located in the request chain to set service_id in the request header based on the endpoint identified by the incoming request. Once this handler is used, there is no need for the original client to pass the service_id header anymore. 

```yaml
# indicate if PathServiceHandler is enabled or not
enabled: true
# mapping from request endpoints to serviceIds
mapping:
  /v1/address/{id}@get: party.address-1.0.0
  /v2/address@get: party.address-2.0.0
  /v1/contact@post: party.contact-1.0.0

```

## pathPrefixService.yml

This is the config file for [PathPrefixServiceHandler][] which is located in the request chain to set service_id in the request header based on the endpoint identified by the incoming request. Once this handler is used, there is no need for the original client to pass the service_id header anymore. This is an alternative option for [PathServiceHandler][]. 

```yaml
# indicate if PathPrefixServiceHandler is enabled or not
enabled: true
# mapping from request path prefixes to serviceIds
mapping:
  /v1/address: party.address-1.0.0
  /v2/address: party.address-2.0.0
  /v1/contact: party.contact-1.0.0

```

## header.yml

The [RouterProxyClient][] requires that both service_id and env_tag must be in the header of the incoming request to do the service discovery. The PathServiceHandler can be used to set the service_id based on the endpoint identified. If you want to set the default env_tag for the environment, you can use the generic HeaderHandler to do so. Here is an example of header.yml config file. 

```yaml
# Enable header handler or not, default to false
enabled: true
# Request header manipulation
request:
  # Remove all the headers listed here
  # remove:
  # - header1
  # - header2
  # Add or update the header with key/value pairs
  # Although HTTP header supports multiple values per key, it is not supported here.
  update:
    env_tag: dev

# Response header manipulation
# response:
  # Remove all the headers listed here
  # remove:
  # - header1
  # - header2
  # Add or update the header with key/value pairs
  # Although HTTP header supports multiple values per key, it is not supported here.
  # update:
  #   key1: value1
  #   key2: value2

```


## Other default handlers

There are some other default handlers that are plugged into the light-router. And normally, 
you shouldn't need to touch these config files unless you know what you are doing. 

* audit.yml
* correlation.yml
* traceability.yml
* cors.yml if it is used
* limit.yml if it is used

[keystore truststore]: /tutorial/security/keystore-truststore/
[security]: /concern/security/
[server]: /concern/server/
[secret]: /tutorial/security/encrypt-decrypt/
[RouterProxyClient]: /service/router/proxy-client/
[PathServiceHandler]: /service/router/path-service/
[PathPrefixServiceHandler]: /service/router/path-prefix-service/
