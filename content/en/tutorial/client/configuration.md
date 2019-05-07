---
title: Setting Client Configuration
linktitle: Setting Client Configuration 
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: "Configuring the client module to customize integration criteria."
weight: 20
sections_weight: 20
draft: false
toc: true
---

## Introduction

The client module will require 3 configuration files to setup integration into the services:

- `client.yml`
- `secret.yml`
- `service.yml`

For the files to be picked up automatically they should all be placed in the resources root or within the `config` sub-folder, or externalized
and passed in through with the `-Dlight-4j-config-dir=/config/` flag.

## client.yml

The `client.yml` file will include configuration for `tls` configuration, as well as `oauth` integration.

### tls
The `tls` configuration will be required for when the client will be reaching out to a service using `tls`. 
```yaml
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: tls/client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: tls/client.keystore
```
It should be noticed that the values of `keystore` and `truststore` represent the paths of them. They should be relative to the external config directory. It means that the absolute path of above truststore and keystore should be `config/tls/client.truststore` and `config/tls/client.keystore` representively.
Otherwise, if all config files flattened into one folder. The configuration could be simplified as:
```yaml
tls:
   ...
   trustStore: client.truststore
   ...
   keyStore: client.keystore
   ...
```

### oauth
The `oauth` configuration will specify all the endpoints and parameters required to communicate with a service protected by oauth2.

```yaml
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

## secret.yml

The `secret.yml` file will contain all the private/secret information for the communication with the service. For example the passwords
to the truststore/keystore, client secrets for oauth2 communication, consul token, etc...

```yaml
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

# Fresh token client secret for OAuth2 server
refreshTokenClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# De-Reference access token to JWT token client secret.
derefClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: token1

# EmailSender password default address is noreply@lightapi.net
emailPassword: change-to-real-password
```


## service.yml

The `service.yml` file provide ioc capability for providing the correct objects for the type of integrations that 
the client will use. Ie. whether a consul client will be used, what type of load balancing, etc..

```yaml
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
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
```


## consul.yml

With the above service.yml config, a consul.yml is needed to control the consul client. Here is an example. 

```
# Consul URL for accessing APIs
consulUrl: https://localhost:8500
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
```
