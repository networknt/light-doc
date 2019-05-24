---
title: "IP Whitelist"
date: 2018-08-29T08:03:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

API security is very crucial, and it is an essential part of the cross-cutting concerns provided by the light platform. For business API endpoints, we should use OAuth 2.0 JWT token to secure them. For some other functional endpoints that cannot be secured by OAuth 2.0, the Basic Authentication is an option. For example, one of the users is using Basic Authentication to security their API deployed on an IoT device. 

In release 1.5.18, we have provided a new feature that allows multiple chains to be defined in a single API instance. Naturally, the /server/info and /health endpoints are separated from the business endpoints so that they can use different middleware handler chains. It has been asked by our customers for a long time to bypass the OAuth 2.0 security check for health check endpoint so that the orchestration layer can access it without providing a token. In most deployment, Consul is used for service registry, and discovery and the /health is periodically accessed from Consul to ensure the service is up and running. It is not very convenient to use JWT or even Basic Authentication in this use case. Since most Consul servers are deployed in physical machines or VMs with fixed IP addresses. It would be great if we can support the IP whitelist for the /health endpoint. After several hours of hard work, the ip-whitelist middleware handler is born. 

### Requirement

* The IP whitelist must be defined per endpoint which is the path and method combination. 
* Both IPv4 and IPv6 should be supported. 
* Need to support the different format of IP addresses. For example, exact, wildcard, slash. 
* User can specify if access is allowed for the endpoints without IP whitelist defined. 

### Implementation

The implementation is a middleware handler and it can be plugged into any API with whitelist.yml and handler.yml configuration files.

Majority of the logic is in the WhitelistConfig class which is responsible for loading the configuration and parse the rules into an object that can be leveraged during the runtime. 

Since we are supporting both IPv4 and IPv6, the address must be parsed with a pattern to differentiate the format. When checking the ACLs in the handler, both IPv4 rules and IPv6 rules will be iterated. 

### Configuration

Here is an exmaple for the whitelist.yml file. The default config in the module has enabled set as false. So you have to externalize it in order to use it in your own application. 

```
# IP Whitelist configuration
---
# Indicate if this handler is enabled or not
enabled: true

defaultAllow: true

paths:
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

### Error Messages

If the whitelist ACL is allowed, then the next handler will be called. Otherwise, an error status will be returned to the caller. 

```
ERR10049:
  statusCode: 403
  code: ERR10049
  message: INVALID_IP_FOR_PATH
  description: Peer IP %s is not in the whitelist for the endpoint %s

```

