---
title: "Docker Deploy"
date: 2019-12-01T23:15:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [mocked][] one of the handlers to return role and client dropdown menus. We have tested locally with curl, and it works. In this step, we are going to dockerize the application and deploy it to the test cloud so that anybody can access it from the Internet. 

### Build Docker Image

To build the docker image and publish it to the hub.docker.com, use the following command from the project root folder. 

The build.sh in the generated project might not be executable on Linux. In that case, you need to change it to executable. 

```
cd ~/networknt/light-reference
chmod +x build.sh
```

To build image and publish.

```
./build.sh 1.0.0
```

Once this is done, a docker image is published to the hub.docker.com, and you can find it [here](https://hub.docker.com/repository/docker/networknt/com.networknt.reference-1.0.0). 


### Deployment

In this step, we are going to deploy the light-reference docker image to the test2 server in the test cloud environment with docker-compose. The service will be behind the light-portal router server, and it needs to be registered to the Consul cluster in order for the light-router to be discovered. 

It requires some special configuration instead of default built-in configuration files. To make things easier, we are going to create a docker-compose-test2.yml as the following in the light-docker repository.

```
# test2 docker-compose
version: "2"
services:
  # There is only one test instance of light-reference and it is allocated at test2
  reference:
    image: networknt/com.networknt.reference-1.0.0:latest
    volumes:
      - ./test2/reference:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host

```

As you can see, there is only one service available for now, and the externalized configuration folder is mapped to the test2/reference folder. 

The following are the configuration files to enable the Consul registry.

client.truststore contains the Consul certificate to allow the light-reference to connect to it with HTTPS. 

client.yml 

```
# This is the configuration file for Http2Client.
---
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: false
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

This file is a copy of the default client.yml with verifyHostname set to false. The Consul cluster on the test cloud only has IP addresses to connect to and cannot verify the domain name from the certificate. 

consul.yml

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
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
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2
enableHttp2: true
```

This is pointing to the right consulUrl with consulToken.

service.yml 

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

```

This file contains the ConsulClientImpl to connect to the Consul cluster for the registry. 

values.yml

```
server.httpsPort: 2498
server.enableRegistry: true
```

This file overwrites two configuration variables in the server.yml packaged into the docker image. It enables the registry and set the HTTPS port to 2498. 

logback.xml is the default one, and it can be updated to turn on the debug if needed. 

All the above files can be found in the [light-docker](https://github.com/networknt/light-docker/tree/master/test2/reference) repository. 

To start the docker-compose-test2.yml on the test2 server, you need to run `ssh test2` from your terminal. Once you are on the remote server, clone the light-docker repository to the networknt folder under the home directory. 


```
ssh test2
cd ~/networknt
git clone git@github.com:networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-test2.yml up -d
```

This will start the docker instance, and you can find it is registered to the Consul from the test cloud Consul cluster. 

In the next step, we are going to update the light-portal [router config][] to allow the /rdata/{host} request to route to the light-reference instance deployed on the test2 server through Consul discovery. 


[mocked]: /tutorial/rest/openapi/light-reference/mock-handler/
[router config]: /tutorial/rest/openapi/light-reference/router-config/
