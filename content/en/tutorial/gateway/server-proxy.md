---
title: "Server Proxy"
date: 2022-04-08T15:41:42-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This tutorial assumes that we build the petstore API with .NET, Python, or Nodejs without any cross-cutting concerns. Let's put the light-gateway instance on the localhost to access the petstore API to address all the cross-cutting concerns. 

### Petstore Serivce

First, let's start the Petstore API and ensure it is working. The default config should have the petstore service started at 9443 with HTTPS/2. 

```
cd ~/networknt/light-example-4j/rest/petstore-maven-single
java -jar target/server.jar
```

Second, let's send a request to the petstore server to ensure it is working.

```
curl -k https://localhost:9443/v1/pets
```

The following result is expected. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

### Server Proxy

Assume we have the gateway built already; we can start it with the default configuration. 

```
cd ~/networknt/light-gateway
java -jar -Dlight-4j-config-dir=config/server-proxy-petstore target/light-gateway.jar
```

The proxy server is started with HTTPS/2 on port 7443. Let's send a request through the proxy server.

```
curl -k https://localhost:7443/v1/pets
```

The same result will be back. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

### Configuration

The configuration files are located in the light-gateway/config/server-proxy-petstore folder.

##### openapi.yaml

To support the scope verification of JWT token and OpenApi validation, we have copied the petstore specification openapi.yaml into the config folder.

##### handler.yml

Here is the default chain of handlers. The last handler is the proxy and it will only be called when all other handlers are passed with cross-cutting concerns addressed. 

```
chains:
  default:
    - exception
    - metrics
    - limit
    - traceability
    - correlation
    - killapp
    - latency
    - memory
    - exchaos
    - cors
    - header
    - body
    - specification
    - security
    - audit
    - sanitizer
    - validator
    - proxy

```

##### values.yml

The values.yml will overwrite some of the properties in the config files. 

* server.yml

We have changed the httpsPort to 7443 and give this service a unique serviceId.

* proxy.yml

We have updated the proxy.host with https://localhost:9443 which is the petstore store API.

* header.yml

We added a customized response header. 

* cors.yml

We have enabled it with the default configuration.

* limit.yml

This is disabled as we don't need for this server-proxy. Our petstore is very fast :)

* sanitizer.yml

It is enabled to do the header html-content encoding. 

* audit.yml

It is enabled.

* client.yml

We defined the tokenKeyServerUrl and tokenKeyUri to get the jwk from the OAuth 2.0 server to verify the JWT token.

* security.yml

To simplify the test, we disable the JWT token verification for now. If you have OAuth 2.0 provider, you can hook it up. 

The following is the entire values.yml


```
# server.yml
server.httpsPort: 7443
server.serviceId: com.networknt.server-proxy-1.0.0

# router.yml
# router.maxRequestTime: 3000

# proxy.yml
proxy.host: https://localhost:9443

# header.yml
header.enabled: true
header.response.update:
  Content-Security-Policy: default-src 'none'; form-action 'none'; frame-ancestors 'none'; img-src 'self' https://images.example.com;

# cors.yml
cors.enabled: true

# limit.yml
limit.enabled: false
limit.rateLimit: 10/s
limit.errorCode: 429

# sanitizer.yml
sanitizer.enabled: true
sanitizer.bodyEnabled: false
sanitizer.headerEnabled: true
sanitizer.headerEncoder: html-content
#sanitizer.headerAttributesToEncode: x-traceability-id
#sanitizer.headerAttributesToIgnore: test

# audit.yml
audit.enabled: true
audit.mask: true
audit.statusCode: true
audit.responseTime: true
audit.auditOnError: false
audit.logLevelIsError: false

# client.yml
client.verifyHostname: false
client.tokenKeyServerUrl: https://localhost:6886
client.tokenKeyUri: /oauth2/keys
client.tokenProxyHost:
client.tokenProxyPort:
client.timeout: 1000
client.tokenKeyEnableHttp2: true
#client.tokenCcClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.tokenRtClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.tokenKeyClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.signClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.signKeyClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.derefClientId: f7d42348-c647-4efb-a52d-4c5787421e73
#client.injectOpenTracing: true

# security.yml
security.enableVerifyJwt: false
security.enableExtractScopeToken: false
security.enableVerifyScope: false
security.keyResolver: JsonWebKeySet
security.bootstrapFromKeyService: true


# service.yml
service.singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          parameters:
            com.networknt.petstore-1.0.0: https://localhost:9443
            Migration: https://lightgateway-dev.networknt.com:8443
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
  - com.networknt.utility.Decryptor:
      - com.networknt.decrypt.AESDecryptor

# pathPrefixService.yml
# pathPrefixService.enabled: true
# pathPrefixService.mapping:
#   /router/dev/de-asia-ekyc-service: de-asia-ekyc-service-1.0.0

# whitelist.yml
whitelist.enabled: false
whitelist.paths:
  # For a particular endpoint(path@method), there are three IPs can access
  '/health/com.networknt.petstore-1.0.0@get':
    # IPv4 Exact
    - '127.0.0.1'
    # IPv4 Wildcard
    - '10.10.*.*'
    # IPv4 Slash
    - '127.0.0.48/30'

  # For a path, the following IP can access regardless the method
  '/prometheus@get':
    # IPv6 Exact
    - 'FE45:00:00:000:0:AAA:FFFF:0045'
    # IPv6 Prefix
    - 'FE45:00:00:000:0:AAA:FFFF:*'
    # IPv6 Slash
    - 'FE45:00:00:000:0:AAA:FFFF:01F4/127'

  # For a particular endpoint(path@method), there are three IPs can access
  '/data@get':
    # IPv4 Exact
    - '127.0.0.2'
    # IPv4 Wildcard
    - '10.10.*.*'
    # IPv4 Slash
    - '127.0.0.48/30'

```
