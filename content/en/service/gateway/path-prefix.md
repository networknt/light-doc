---
title: "Path Prefix Service"
date: 2022-11-20T10:01:13-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For the details of the middleware handler, please visit the [PathPefixService][] document. 

The PathPrefixService handler is one of the middleware handlers that are used to identify the service_id before calling the router handle to route the traffic to multiple downstream APIs. 

This handler is simple without depending on the OpenAPI specification in the light gateway(LG) and light client proxy(LCP). It is suitable when all the backend APIs have a unique prefix in the request path. 

For the handler configuration, please visit the [PathPefixService][] document. The following is for the values.yml in a shared gateway with all other related configurations for the router feature.

Once you have configured the PathPrefixServiceHandler, you need to find a way to map the service_id in the header to a real host for the downstream API. There are several ways to do that with service discovery. If you are using light-portal for the registry and discovery, you can set up the service.yml with PortalRegistry. 

If you know the static URLs for the downstream API, you can use the DirectRegistry in the service.yml for static routing. This can be easily converted to service discovery in the future. 

Here is the example for the pathPrefixService.mapping. 

```
pathPrefixService.mapping:
  /v1: com.networknt.petstore-1.0.0
  /gateway/Migration/1.0.0: Migration

```

Here is the example for the service.yml with DirectRegistry.

```
service.singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          protocol: https
          host: localhost
          port: 7080
          path: direct
          parameters:
            com.networknt.petstore-1.0.0: https://localhost:9443
            Migration: http://dev-webservices.networknt.com
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
  - com.networknt.utility.Decryptor:
      - com.networknt.decrypt.AESDecryptor

```

You can see there are two serviceIds that map to two different hosts in the parameters. 

For the router.yml, we might need to rewrite the URL to the path that the target server specification defined. 


```
# router.yml
router.maxRequestTime: 5000
router.urlRewriteRules:
  - /v1/market(.*)$ /dev-test-ns/market/v1/market$1
  - /v1/pets(.*)$ /dev-test-ns/petstore/v1/pets$1
  - /gateway/Migration/1.0.0(.*)$ /1.0.0/Migration$1

```

For a running example, you can find it at https://github.com/networknt/light-gateway/tree/master/config/shared-gateway


[PathPefixService]: /concern/path-prefix-service/

