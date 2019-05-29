---
title: "Register Service"
date: 2019-04-16T05:17:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [deployed service2][] to the same server instance and tested both services. In a real production environment, the server instance will be deployed to the could with no static IP and port number. All services need to register itself to the registry like Consul to let client to discovery them. 

Unlike other frameworks, all endpoints are in the same server instance, light-hybrid-4j services can share the same instance and each server instance might have differnet number of services depending how each service is scaled. This requires that each individual service need to register itself instead of the server instace itself. 

The RpcStartupHookProvider is responsible to register the service right after the service is discovered and merged during the server startup. It simply tell the Server instance that a list of services need to be registered and the Server will help to register them after the binding is done. 

For this step, there is no code change but only configuration changes. It shows users how to update the configuration of the light-4j platform to change the behavior of the application. 

First, we need to create a config folder under hello-world folder. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world
mkdir config
```

Now we need to copy several configuration files into this folder. 

### client.keystore and client.truststore 

They are binary files and the client.truststore contains the certificate for our Consul cluster. Regarding how this client.truststore is created, please refer to https://doc.networknt.com/tutorial/common/discovery/consul-tls/

### client.yml

To make it simple, we are going to use the IP address to access the Consul note1. So, we need to disable the verfiyHostname in the default client.yml to set `verifyHostname: false`. As we are using one-way TLS, we only need to set `loadTrustStore: true`.


```
# This is the configuration file for Http2Client.
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: false
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

### consul.yml

This file constains all the parameters for consul registy. 

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
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

### secret.yml

The secret.yml contains the consul token to access the consul server for registry.

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

### server.yml

In the server config, let's enable the registry and dynamic port.

```

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 49587

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 49588

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.petstore-1.0.1

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

```

### service.yml

In the beginning of this config file, let's add the mappings of service registry and discovery. 

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
      - com.networknt.rpc.router.RpcRouter
  # StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
  - com.networknt.server.StartupHookProvider:
      # registry all service handlers by from annotations
      - com.networknt.rpc.router.RpcStartupHookProvider
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
      # Jwt Token Verification for signature and expiration
      - com.networknt.rpc.security.JwtVerifyHandler
      # SimpleAudit Log important info about the request into audit log
      - com.networknt.audit.AuditHandler

```

### logback.xml

Let's change the logback.xml to enable the debugging to the console. 

```

```


### Start Server

In order to make the command line start easier, let's first copy the two services into a service folder under hello-world. Note that before copying and starting the server, you need to make sure to build server, service1 and service2 with `mvn clean install` first. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world
mkdir service
cd service
cp ../service1/target/service1-1.0.0.jar .
cp ../service2/target/service2-1.0.0.jar .
```

Let's start the server with the externalized config folder. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world
java -cp server/target/hello-1.0.0.jar:service/* com.networknt.server.Server
```

The above command is the same as the previous command in the [deploy service2][]. We just copied service jars into a folder in hello-world instead of /tmp. 

Not let's start the server with the externalized config folder in the hello-world folder.

```
cd ~/networknt/light-example-4j/hybrid/hello-world
java -Dlight-4j-config-dir=config -Dlogback.configurationFile=config/logback.xml -cp server/target/hello-1.0.0.jar:service/* com.networknt.server.Server
```

Now, if you go to the Consol web ui, you can see there are one server instance and for service instances registered on the Consul. 

Here is the list services registered on the Consul.

```
com.networknt.petstore-1.0.1
lightapi.net.service1.info.0.1.0
lightapi.net.service1.query.0.1.0
lightapi.net.service2.command.0.1.0
lightapi.net.service2.command.0.1.1
```

The first entry is the server instance and the rest entries are four services in the following format. 

```
host.service.action.version
```

At this stage, a client application can use the dot to concat host, service, action and version to create a key to lookup on the service registry to find out IP and port in order to establish direct connection to the right service. 



[deployed service2]: /tutorial/hybrid/hello-world/deploy-service2/
[deploy service2]: /tutorial/hybrid/hello-world/deploy-service2/
