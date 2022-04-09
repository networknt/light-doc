---
title: "Client Proxy Server Proxy"
date: 2022-04-08T22:28:07-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

After we have done the [server-proxy][], we can set up the market API to use the client-proxy to access the petstore API. 

Assume that the petstore API and the server-proxy still up and running. The following command can confirm the petstore is working and accessible from the server-proxy.

```
curl -k https://localhost:7443/v1/pets
```

And the result should be.

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

### Client Proxy

Let's start another light-gateway instance for the client-proxy. The market API will call the client-proxy, the client-proxy will call the server-proxy and then the server-proxy will call the petstore API. 

Open another terminal and start the client-proxy. 

```
cd ~/networknt/light-gateway
java -jar -Dlight-4j-config-dir=config/client-proxy-market target/light-gateway.jar
```

### Market API

Let's start the market API.

```
cd ~/networknt/light-example-4j/rest/market
java -jar target/server.jar
```

The server will start and listen to port 6443 on HTTPS/2.

### Test Market

```
curl -k https://localhost:6443/petstore/products
```

you should have the following result.


```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

### Client Proxy Configuration

The configuration files are located in light-gateway/config/client-proxy-market folder.

##### handler.yml

Here is the chain for routing only.

```
chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - cors
    - header
    - router
```

##### values.yml

* server.yml

Set the httpPort to 9080 and give it a serviceId called com.networknt.client-gateway-market

* router.yml

Allow the hostWhitelist for localhost and set the maxRequestTime to 3 seconds.

* pathPrefixService.yml

Add /v1/pets to com.networknt.petstore mapping

* service.yml

Add an entry for petstore to localhost:7443 which is the server proxy of petstore.

```
com.networknt.petstore: https://localhost:7443
```

Here is the entire file.


```
# server.yml
server.httpPort: 9080
server.enableHttp: true
server.enableHttps: false
server.enableHttp2: false
server.serviceId: com.networknt.client-gateway-market

# router.yml
router.maxRequestTime: 3000
router.hostWhitelist:
  - localhost

# pathPrefixService.yml
pathPrefixService.mapping:
  /v1/pets: com.networknt.petstore

# service.yml
service.singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          parameters:
            com.networknt.petstore: https://localhost:7443
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
  - com.networknt.utility.Decryptor:
      - com.networknt.decrypt.AESDecryptor

```



[server-proxy]: /tutorial/gateway/server-proxy/
