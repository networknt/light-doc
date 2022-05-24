---
title: "External Handler"
date: 2022-05-18T16:47:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When using the light-gateway to access external services from the corporate network through a proxy server like McAffee WebGateway, the router handler doesn't support the proxy configuration. 

We are trying to resolve the issue in the client module of light-4j; however, it will take some time, and we have to find a workaround to help some of the projects with tight schedules.

The solution is to create a generic handler with JDK 11 HTTP client to connect to the external services, and it supports the proxyHost and porxyPort configuration. 

As we have put this handler into the default chain, we need to make sure this handler will only be applied for specific paths. 

As we have put this handler into the default chain, we need to make sure this handler will only be applied for specific paths, and it must be before the router handler in the default chain. 

The handler is named ExternalServiceHandler, and it has a configuration file named external-service.yml

Here is the default external-service.yml file provided by the ingress-proxy module.

```
# Configuration for external service handler to access third party services through proxy/gateway.
# Indicate if the handler is enabled or not
enabled: ${externalService.enabled:false}
# Proxy Host if calling within the corp network with a gateway like Mcafee gateway.
proxyHost: ${externalService.proxyHost:}
# Proxy Port if proxy host is used. default value will be 443 which means HTTPS.
proxyPort: ${externalService.proxyPort:}
# If HTTP2 is used to connect to the external service.
enableHttp2: ${externalService.enableHttp2:false}
# A list of request path to the service host mappings. Other requests will skip this handler. The value is
# a string with two parts. The first part is the path and the second is the target host the request is
# finally routed to.
pathHostMappings: ${externalService.pathHostMappings:}

```

In most cases, you should enable the handler and set the path to the host mappings in your values.yml file. 

```
# handler.yml
handler.handlers:
  .
  .
  .
  # add the ExternalServiceHandler to the end of the handler list.
  - com.networknt.proxy.ExternalServiceHandler@external

handler.chains.default:
  .
  .
  .
  - header
  # add the external alias after header and before prefix and router.
  - external
  - prefix
  - router
  .
  .
  .


# external-service.yml
externalService.enabled: true
externalService.pathHostMappings:
  - /sharepoint sharepoint.microsoft.com
  - /abc abc.com

```

In the above values.yml configuration,  we have enabled the handler to set the externalService.enabled to true. 

Also, we added to path to host mappings to the externalService.pathHostMappings list. For each entry, it is a string with two parts. The first part is the path of the request and the second part is the target host the request will be routed. You can set up many mapping based on how many generic external services your internal network accesses.

The handler is a singleton, and it caches the HttpClient and only creates it the first time it is used.  For each HttpRequest, we need to capture the original request and reconstruct a new request to the target server based on the path to host mappings. 

This handler is a simple version of the proxy handler; it is not a full-blown implementation.

Here are some limitations: 

* Only GET/DELETE/POST/PUT/PATCH methods are supported. 
* Only GET/DELETE supports query parameter
* When copying the headers, only the first value is copied.

