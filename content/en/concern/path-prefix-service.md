---
title: "Path Prefix Service"
date: 2022-11-20T10:24:00-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When using the light-gateway, each request must have service_id in the header to allow the router to discover the service before invoking downstream services. We have to do that due to the unpredictable path between services. Suppose you are sure that a unique path prefix can identify all the downstream services. In that case, you can use this Path to ServiceId mapper handler to uniquely identify the service_id and put it into the request header. The client can invoke the service just like it is invoking it directly. To do that, you can leverage the [PathPrefixServiceHandler][] in the egress-router module in the light-4j repository.

Please note that you cannot invoke /adm/health or /adm/server/info endpoints as these are the standard endpoints injected by the framework, and all services will have them on the same path. The
the router cannot figure out which service you want to invoke, and an error message will be returned. In other words, we should only use this middleware handler with business endpoints. 

Unlike [PathServiceHandler][], this handler does not require OpenAPIHandler in the request/response chain. It cannot validate beyond the path prefix as it doesn't have the concept of an endpoint, a combination of the request path and HTTP method.

Previously, this handler would skip the logic when server_url was in the header. However, since we have updated the TokenHandler to support multiple downstream hosts with different OAuth 2.0 servers, we need to put the service_id into the header regardless of whether the server_url is in the header. Also, this handler works on a best-effort basis, so it only works if the prefix is in the config.

This is the most straightforward mapping with the prefix, and all APIs behind the http-sidecar or light gateway should have a unique prefix. All the services should follow this convention to have a unique prefix in the request path to identify itself. 


### Dependency

The following dependency needs to be added to pom.xml in your project to use this handler.

```
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>egress-router</artifactId>
                <version>${version.light-4j}</version>
            </dependency>
```

### Configuration

The following is the default built-in configuation file pathPrefixService.yml

```
# indicate if PathPrefixServiceHandler is enabled or not
enabled: ${pathPrefixService.enabled:true}
# mapping from request path prefixes to serviceIds
mapping: ${pathPrefixService.mapping:}
# The following are examples in the values.yml
#   /v1/address: party.address-1.0.0
#   /v2/address: party.address-2.0.0
#   /v1/contact: party.contact-1.0.0
```

This handler will be enabled by default if it is in the request/response chain. However, the pathPrefixService.mapping is empty by default, so the request will skip this handle and go to the next handler. 

The mapping is a map between the path prefixes to the serviceIds and there are three ways to overwrite the pathPrefixService.mapping in the values.yml file. 

The following is in YAML format for readability. 

values.yml
```
pathPrefixService.mapping:
  /v1/address: party.address-1.0.0
  /v2/address: party.address-2.0.0
  /v1/contact: party.contact-1.0.0
```

The following is in JSON format. It is suitable for the config server as it requires a string for the property value. 

values.yml
```
pathPrefixService.mapping: {"/v1/address": "party.address-1.0.0", "/v2/address": "party.address-2.0.0", "/v1/contact": "party.contact-1.0.0"}
```

The following is in a Java map format. It can also be used for config server. 

values.yml

```
pathPrefixService.mapping: /v1/address=party.address-1.0.0&/v2/address=party.address-2.0.0&/v1/contact=party.contact-1.0.0
```

### Usage

Once you configure this handler in the values.yml file, you need to add it to the request/response chain to allow it to work. 

In the handler.yml or the values.yml if you have externalized the handlers definition and default chain. 


```
handler.handlers:
  .
  .
  .
  - com.networknt.router.middleware.PathPrefixServiceHandler@prefix

handler.chains.default:
  - exception
  - metrics
  - prefix
  .
  .
  .
  - router
  .
  .
  .

```

This handler must be placed before the router handler as it will put the service_id in the request header for the router to find the downstream API host. 

In light-gateway without OpenAPI specification deployed, you have to put the prefix right after the metrics handler so that the auditInfo can be populated for metrics even if security verification is failed. 

For a real example of using this handle in the light gateway, please visit [path-prefix][] in the gateway document. 


[PathPrefixServiceHandler]: https://github.com/networknt/light-4j/blob/master/egress-router/src/main/java/com/networknt/router/middleware/PathPrefixServiceHandler.java
[PathServiceHandler]: https://github.com/networknt/light-4j/blob/master/egress-router/src/main/java/com/networknt/router/middleware/PathServiceHandler.java
[path-prefix]: /service/gateway/path-prefix/

