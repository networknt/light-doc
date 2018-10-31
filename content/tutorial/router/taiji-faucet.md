---
title: "Taiji Faucet light-router deployment"
date: 2018-10-31T11:40:45-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


** This tutorial is working with light-router 1.5.21 image from docker hub. **

Taiji-faucet is an application used to populate Taiji coins to a newly created address for Taiji-Chain testnet. It is a very simple API based on OpenAPI specification. I have recorded two videos to get it started and deployed to a Kubernetes cluster. 

- Start taiji-faucet from specification to code generation

{{< youtube eQ1WjpV3dYo >}}

- Update the configuration to deploy on a Kubernetes cluster with Consul discovery

{{< youtube tcKP-tziADw >}}

In this tutorial, we are going to set up a light-router instance on the portal VM to perform service discovery and later serve a React based SPA and static content. Basically, the light-router is acted as a BFF for the taiji-faucet service running in the cloud. 

We are going to use docker-compose to start the light-router instance, so the first step is to create a config folder for it in https://github.com/networknt/light-config-test/tree/develop/light-router/taiji-faucet

### Docker Compose

First, let's create a docker-compose.yml file in the folder. 

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

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true
```

In the above yml file, we are using the latest version of the light-router image from networknt organization on docker hub. Also, it specified config folder from the current config. 

Now let's create a config folder to hold configuration files for the light-router. 

### router.yml

We need to create a router.yml file. You can copy the default one from the light-router repository and modify it. Here is the final version of this project.

```
---
# Client Router Configuration
# As this router is built to support discovery and security for light-4j services,
# the outbound connection is always HTTP 2.0 with TLS enabled.

# If HTTP 2.0 protocol will be accepted from incoming request.
http2Enabled: true

# If TLS is enabled when accepting incoming request. Should be true on test and prod.
httpsEnabled: true

# Max request time in milliseconds before timeout to the server.
maxRequestTime: 1000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3
```

Note that we have both HTTPS and HTTP2 enabled. 


### client.yml

The light-router will need light-4j Http2Client to do the service lookup on Consul cluster and also invoke downstream service. It might also be responsible for security with interactions to light-oauth2 services. 

Here is the client.yml 

```
# This is the configuration file for Http2Client.
---
bufferSize: 32
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
```

Please note that the bufferSize is set to 32 for this application. If you application handles bigger request body, then you might need to increase the bufferSize. 

Since the Http2Client will need to communicate with Consul and downstream taiji-faucet service in TLS, we need to ensure that the client.truststore contains both consul and taij-faucet certificates. As taiji-faucet is using the default server.keystore, so the certificate is already in the generated client.truststore in light-router. After [adding the consul certificate][], the client.truststore should look like this. 

```
keytool -list --keystore client.truststore
Enter keystore password:  
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 3 entries

consul-ca, 5-Aug-2018, trustedCertEntry, 
Certificate fingerprint (SHA1): 9E:8C:1F:08:59:76:28:90:E3:E5:FF:F8:39:6E:66:55:16:EB:49:28
consul, 3-Aug-2018, trustedCertEntry, 
Certificate fingerprint (SHA1): 4D:62:10:ED:FB:3A:24:90:A5:C9:A2:3A:8D:D3:B3:B6:BB:30:0C:4F
server, 19-Jan-2013, trustedCertEntry, 
Certificate fingerprint (SHA1): 87:3B:92:F5:0A:FE:99:E9:C5:AF:38:F1:C8:65:98:7A:C1:13:19:1D
```

The password for the client.trustore can be found it the secret.yml which is described below. 

### consul.yml

Although the light-router instance doesn't need to register itself to the Consul cluster, it needs to lookup taiji-faucet service from the Consul cluster. So it needs the consul.yml config file. 

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 30s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: true
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
```

In this configuration, only the consulUrl is used. You can see it is an HTTPS URL. 


### secret.yml

To communicate with Consul, we also need to put the consulToken into the secret.yml file.

```
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
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

This file also contains passwords for client.truststore and server.keystore. 


### service.yml

In order to ensure that service discovery works, we need to inject several implementations for discovery. Here is the final copy of the service.yml file. The first section until HandlerProvider is for Consul registry and discovery. 

```
# Singleton service factory configuration/IoC injection
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: consul
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
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
  # Direct requests to named services based on the request path
  - com.networknt.router.middleware.PathPrefixServiceHandler
```

### pathPrefixService.yml

To map the path to a service, we need to enable PathPrefixServiceHandler in the above service.yml file. Also, we need a specific config file call pathPrefixService.yml to be created. 

```
# Whether to enable PathPrefixServiceHandler
enabled: true

# mapping from request path prefixes to serviceIds
mapping:
  /faucet: io.taiji.faucet-1.0.0
```

Please note that the above configuration means that any request with /faucet will be routed to io.taiji.faucet-1.0.0 service in the Kubernetes cluster. 

### server.yml

This is to control the server instance. 

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort: 8080

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort: 8443

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled. When Http2 is enable, enableHttps is true and enableHttp is false by default.
# If you want to have http enabled, enableHttp2 must be false.
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.router-0.1.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1
```

As you can see that the light-router server is listening to 8443 for HTTPS/2. The port number matches the port mapping in the docker-compose.yml in the parent folder.

As this light-router is public-facing, we have created a server.keystore with certs from Let's Encrypt. If you want to learn the process, please refer to [security tutorial][]


[adding the consul certificate]: /tutorial/common/discovery/consul-tls/
[security tutorial]: /tutorial/security/

