---
title: "Compose Consul"
date: 2018-11-04T12:14:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the [previous step][], we have shown how to use docker-compose with Consul for service registry and discovery. To make the use case simple and let users focus on the registry and discovery only, we used a docker instance of the Consul with only HTTP support. Also, we didn't explore host network in the docker-compose and dynamic port allocation in the server.yml 

In this tutorial, we are going to use the chain-write and chain-reader service for taiji-chain to show you what production-like configuration should be if you don't want to use Kubernetes but stay with docker-compose or docker-swarm for your target deployment environment. 

### Prerequisites

You need to have a [Consul cluster installed][] with at least three nodes and HTTPS/2 must be enabled. 

[DOCKER_HOST_IP must be set][] as an environment on your deployment server.


### docker-compose.yml

The real deployment has three nodes, and each has two services. In this tutorial, we are focusing on only one node test3.

Here is the docker-compose file. 

```
# testnet test1 docker-compose
version: "2"
services:
  chainwriter:
    image: networknt/com.networknt.chainwriter-1.0.0:latest
    # ports:
    #   - 8443:8443
    volumes:
      - ./test3/chain-writer:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host
    # networks:
    #   - localnet
  chainreader:
    image: networknt/com.networknt.chainreader-1.0.0:latest
    # ports:
    #   - 8444:8444
    volumes:
      - ./test3/chain-reader:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
    # networks:
    #   - localnet

# Networks
#
# networks:
#   localnet:
#     external: true

```

We have commented out the ports and set the network_mode to host. Also, the localnet is removed as it is not used. Please be aware that we pass in an environment variable to the container named STATUS_HOST_IP with the value from DOCKER_HOST_IP env variable. 

### consul.yml

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

Please note that HTTPS is used in the consulUrl and httpCheck is enabled to allow Consul to callback to service's health check endpoint with serviceId. 

To connect to Consul cluster with HTTPS, we need to put the consul certificates into the client.truststore. Here is the content to of the client.truststore 

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

### secret.yml

Update the secret.yml to add consul token. 

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

# Fresh token client secret for OAuth2 server
refreshTokenClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
# consulToken: the_one_ring
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527

# EmailSender password
emailPassword: change-to-real-password

```

### server.yml

We need to update the server.yml to enable registry and enable dynamic port. 

```

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 8080

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 8444

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
serviceId: com.networknt.chainreader-1.0.0

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
environment: 0002

# Build Number
# Allows teams to audit the value and set it according to their release management processes
buildNumber: 1.0.0

```

### service.yml

Update service.yml to enable Consul registy and discovery.

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
  - com.networknt.kafka.ChainConsumer:
    - com.networknt.kafka.LedgerConsumer
  - com.networknt.kafka.ChainStreams:
    - com.networknt.kafka.ChainReaderStreams
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
  - com.networknt.server.StartupHookProvider:
    - com.networknt.chainreader.ReaderStartupHookProvider
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
  - com.networknt.server.ShutdownHookProvider:
    - com.networknt.chainreader.ReaderShutdownHookProvider

```

### Summary

Once all configuration is done, you can start the docker-compose and see the services registered on the Consul.

[previous step]: /tutorial/common/discovery/docker/
[Consul cluster installed]: /tutorial/common/discovery/consul-tls/
[DOCKER_HOST_IP must be set]: /tutorial/eventuate/getting-started/
