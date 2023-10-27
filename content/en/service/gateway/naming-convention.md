---
title: "Path Prefix Naming Convention"
date: 2023-10-20T15:09:17-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

A better path prefix naming convention can help you manage the gateway more easily when using light-gateway as a centralized shared gateway for many APIs, especially when you have two or more gateways deployed in the DMZ and Corporate network. 

### Path Prefix

The path prefix is the beginning of the path in the request URL. In the gateway usage, it can be used to identify the gateway instance, API id and API version. 


For example, the following URL has the path prefix "/gateway/account/v1" for https://gateway-dev.networknt.com/gateway/account/v1/personal/322


### Naming Convention

When an API is onboarding with the light gateway, we need to define the path prefix for it, as many configurations associated with the API are based on the path prefix to identify the API. 

The path prefix should have three sections to identify the gateway name, API identifier and API version. 

<gateway>/<API identifier>/<API version>

All three parts should be in lowercase. 

If you have only one centralized gateway, you can just use the "gateway" as the gateway name. Otherwise, use the corresponding name, like corpgateway, etc. 

The API identifier is used to identify the API service. It must be unique in combination with the API version. 

The API version is used to identify the major version of the API service, and it must be preceded by the letter "v".


### URL Rewrite

The path prefix might be rewritten in the gateway configuration to make sure that the right path is sent to the downstream API with router or proxy handlers. 

For example, the incoming request path prefix can be. 

```
/gateway/account/v1(.*)$
```

It can be rewritten as. 

```
/dev-k8s-namespace/account/v1$1
```

Example in values.yml configuration. 

```
router.urlRewriteRules:
/gateway/account/v1(.*)$ /dev-k8s-namespace/account/v1$1
```

