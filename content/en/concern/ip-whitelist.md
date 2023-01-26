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
reviewed: true
---

API security is very crucial, and it is an essential part of the cross-cutting concerns provided by Light Platform. For business API endpoints, we should use the OAuth 2.0 JWT token to secure them. For some other functional endpoints that cannot be secured by OAuth 2.0, Basic Authentication or API Key is an option. For example, one of the users is using Basic Authentication to secure their API deployed on an IoT device.

In release 1.5.18, we provided a new feature that allows multiple chains to be defined in a single API instance. Naturally, the admin endpoints, like /adm/server/info and /adm/chaosmonkey endpoints, are separated from the business endpoints so that they can use different middleware handler chains. It has been asked by our customers for a long time to bypass the OAuth 2.0 security check for a health check endpoint so that the orchestration layer can access it without providing a token. In most deployments, Consul is used for service registry, and discovery and the /health is periodically accessed from Consul to ensure the service is up and running. It is not very convenient to use JWT or even Basic Authentication in this use case. Since most Consul servers are deployed in physical machines or VMs with fixed IP addresses. It would be great if we can support the IP whitelist for the /health endpoint. After several hours of hard work, the ip-whitelist middleware handler was born.
 
### Requirements

* The IP whitelist must be defined per path prefix which is the beginning of the request path. 
* Both IPv4 and IPv6 should be supported. 
* Need to support the different format of IP addresses. For example, exact, wildcard, slash. 
* Users can specify if access is allowed for the path prefixes without IP whitelist defined.

### Implementation

The implementation is a middleware handler and it can be plugged into any API with whitelist.yml and handler.yml configuration files.

The majority of the logic is in the WhitelistConfig class which is responsible for loading the configuration and parsing the rules into an object that can be leveraged during the runtime.

Since we are supporting both IPv4 and IPv6, the address must be parsed with a pattern to differentiate the format. When checking the ACLs in the handler, both IPv4 rules and IPv6 rules will be iterated. 

### Configuration

Here is the built-in whitelist.yml file in the module resource/config folder that will be used by default. All the properties can be overwritten with the values.yml in your application. For the whitelist.paths, we have added two examples in both YAML and JSON formats. 

```
# IP Whitelist configuration
---
# Indicate if this handler is enabled or not. It is normally used for the third party integration
# so that only approved IPs can connect to the light-gateway or http-sidecar at certain endpoints.
# The default value is true, and it will take effect as long as this handler is in the chain.
enabled: ${whitelist.enabled:true}
# Default allowed or denied if there is no rules defined for the path or the path is not defined.
# If defaultAllow is true, all IP addresses defined under the matched path will be allowed and IP
# addresses are not in the list will be denied. If the request path prefix is not defined, then all
# requests will be allowed by default. Basically, only IPs not listed under a path will be denied.
# If defaultAllow is false, all IP addresses defined under the matched path will be denied and IP
# addresses are not in the list will be allowed. If the request path prefix is not defined, then all
# requests will be denied by default. Basically, only IPs not listed under a path will be allowed.
defaultAllow: ${whitelist.defaultAllow:true}
# List of path prefixes and their access rules. It supports IPv4 and IPv6 with Exact, Wildcard and
# Slash format. The path prefix is defined as request path prefix only without differentiate method.
paths: ${whitelist.paths:}
# The following format is the YAML format suitable for externalized values.yml in local filesystem.
#  /health/com.networknt.petstore-1.0.0:
#    - 127.0.0.1
#    - 10.10.*.*
#    - 127.0.0.48/30
#  /prometheus:
#    - FE45:00:00:000:0:AAA:FFFF:0045
#    - FE45:00:00:000:0:AAA:FFFF:*
#    - FE45:00:00:000:0:AAA:FFFF:01F4/127
#  /data:
#    - 127.0.0.2
#    - 10.10.*.*
#    - 127.0.0.48/30

# The following format is the JSON format suitable for both local values.yml and config server.
# paths: {"/health/com.networknt.petstore-1.0.0":["127.0.0.1","10.10.*.*","127.0.0.48/30"],"/prometheus":["FE45:00:00:000:0:AAA:FFFF:0045","FE45:00:00:000:0:AAA:FFFF:*","FE45:00:00:000:0:AAA:FFFF:01F4/127"],"/data":["127.0.0.2","10.10.*.*","127.0.0.48/30"]}

```

Here is an exmaple for the values.yml file in YAML format. 

```
# whitelist.yml

whitelist.paths:
  # For a particular path prefix, there are three IPs can access
  /health/com.networknt.petstore-1.0.0:
    # IPv4 Exact
    - 127.0.0.1
    # IPv4 Wildcard
    - 10.10.*.*
    # IPv4 Slash
    - 127.0.0.48/30

  # For a path, the following IP can access regardless the method
  /prometheus:
    # IPv6 Exact
    - FE45:00:00:000:0:AAA:FFFF:0045
    # IPv6 Prefix
    - FE45:00:00:000:0:AAA:FFFF:*
    # IPv6 Slash
    - FE45:00:00:000:0:AAA:FFFF:01F4/127

  # For a particular path, there are three IPs can access
  /data:
    # IPv4 Exact
    - 127.0.0.2
    # IPv4 Wildcard
    - 10.10.*.*
    # IPv4 Slash
    - 127.0.0.48/30
```

Here is an example for values.yml file in JSON format.

```
# whitelist.yml

whitelist.paths: {"/health/com.networknt.petstore-1.0.0":["127.0.0.1","10.10.*.*","127.0.0.48/30"],"/prometheus":["FE45:00:00:000:0:AAA:FFFF:0045","FE45:00:00:000:0:AAA:FFFF:*","FE45:00:00:000:0:AAA:FFFF:01F4/127"],"/data":["127.0.0.2","10.10.*.*","127.0.0.48/30"]}

```

Please note that the defaultAllow is true in the built-in configuration file whitelist.yml for the module. For most organizations, this configuration should meet the requirement. However, if you want more security protection, you can set up defaultAllow to false in the values.yml file. Here are the rules for defaultAllow. 

```
Default allowed or denied if there is no rules defined for the path or the path is not defined.

If defaultAllow is true, all IP addresses defined under the matched path will be allowed, and IP
addresses that are not in the list will be denied. If the request path prefix is not defined, then all
requests will be allowed by default. Basically, only IPs not listed under a path will be denied.

If defaultAllow is false, all IP addresses defined under the matched path will be denied, and IP
addresses that are not in the list will be allowed. If the request path prefix is not defined, then all
requests will be denied by default. Basically, only IPs not listed under a path will be allowed.

```

To understand how the rules work, you can check the test cases in the [repo](https://github.com/networknt/light-4j/tree/master/ip-whitelist/src/test/java/com/networknt/whitelist). 

### Error Messages

If the whitelist ACL is allowed, then the next handler will be called. Otherwise, an error status will be returned to the caller. 

```
ERR10049:
  statusCode: 403
  code: ERR10049
  message: INVALID_IP_FOR_PATH
  description: Peer IP %s is not in the whitelist for the endpoint %s

```

