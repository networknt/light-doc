---
title: "Rate Limit"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Although our framework can potentially handle millions of requests per second, some public-facing APIs might be a good idea to enable this handler to limit the concurrent requests to a certain level to avoid DDoS attacks. 

When the http-sidecar, light-router or light-proxy is used inside a corporate network for a slow backend API, this handler can be used to throttle the requests to protect the backend API from being overwhelmed. 


As this handler will impact the overall performance a little bit, it is not configured as default in the [light-codegen](https://github.com/networknt/light-codegen). You must config the feature as true in your light-codegen config.yaml/config.json or update the handler.yml config later in the deployment phase.

### Dependency

To use the handler, the following dependency needs to be added to pom.xml in your project.

1.6.x release with JDK8
```
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>limit</artifactId>
                <version>${version.light-4j}</version>
            </dependency>
```

2.0.x release with JDK11

```
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>rate-limit</artifactId>
                <version>${version.light-4j}</version>
            </dependency>
```


### handler.yml

As this middleware is not plugged in by default, we need to add it to the handler.yml in the config folder. As this rate-limiting handler needs to be failed fast, it needs to be put right after ExceptionHandler and MetricsHandler. The reason it is after MetricsHandler is to capture 503 or 429 error codes in InfluxDB and Grafana for monitoring on production.


```
handlers:
  # Rate limit middleware to prevent DDoS attacks externally or throttle requests internally
  - com.networknt.limit.LimitHandler@limit
.
.
.
.
.
.

chains:
  default:
    - exception
    - metrics
    - limit
    - traceability
    - correlation

```

### Config

Here is the configuration file for rate limiting.

```yaml
---
---
# Rate Limit Handler Configuration

# If this handler is enabled or not. It is disabled by default as this handle might be in
# most http-sidecar, light-proxy and light-router instances. However, it should only be used
# internally to throttle request for a slow backend service or externally for DDoS attacks.
enabled: ${limit.enabled:false}
# Maximum concurrent requests allowed per second on the entire server. This is property is
# here to keep backward compatible. New users should use the rateLimit property for config
# with different keys and different time unit.
concurrentRequest: ${limit.concurrentRequest:2}
# This property is kept to ensure backward compatibility. Please don't use it anymore. All
# requests will return the rate limit headers with error messages after the limit is reached.
queueSize: ${limit.queueSize:-1}
# If the rate limit is exposed to the Internet to prevent DDoS attacks, it will return 503
# error code to trick the DDoS client/tool to stop the attacks as it considers the server
# is down. However, if the rate limit is used internally to throttle the client requests to
# protect a slow backend API, it will return 429 error code to indicate too many requests
# for the client to wait a grace period to resent the request. By default, 503 is returned.
errorCode: ${limit.errorCode:503}
# Default request rate limit 10 requests per second and 10000 quota per day. This is the
# default for the server shared by all the services. If the key is not server, then the
# quota is not applicable.
# 10 requests per second limit and 10000 requests per day quota.
rateLimit: ${limit.rateLimit:10/s 10000/d}
# Key of the rate limit: server, address, client, user
# server: The entire server has one rate limit key, and it means all users share the same.
# address: The IP address is the key and each IP will have its rate limit configuration.
# client: The client id in the JWT token so that we can give rate limit per client.
# user: The user id in the JWT token so that we can set rate limit and quota based on user.
key: ${limit.key:server}
# If server is the key, we can set up different rate limit per request path prefix.
server: ${limit.server:}
# If address is the key, we can set up different rate limit per address and optional per
# path or service for certain addresses. All other un-specified addresses will share the
# limit defined in rateLimit.
address: ${limit.address:}
# If client is the key, we can set up different rate limit per client and optional per
# path or service for certain clients. All other un-specified clients will share the limit
# defined in rateLimit. When client is select, the rate-limit handler must be after the
## JwtVerifierHandler so that the client_id can be retrieved from the auditInfo attachment.
client: ${limit.client:}
# If user is the key, we can set up different rate limit per user and optional per
# path or service for certain users. All other un-specified users will share the limit
# defined in rateLimit. When user is select, the rate-limit handler must be after the
# JwtVerifierHandler so that the user_id can be retrieved from the auditInfo attachment.
user: ${limit.user:}
# Client id Key Resolver.
clientIdKeyResolver: ${limit.clientIdKeyResolver:com.networknt.limit.key.JwtClientIdKeyResolver}
# Ip Address Key Resolver.
addressKeyResolver: ${limit.addressKeyResolver:com.networknt.limit.key.RemoteAddressKeyResolver}
# User Id Key Resolver.
userIdKeyResolver: ${limit.userIdKeyResolver:com.networknt.limit.key.JwtUserIdKeyResolver}

```

- Update enabled to true to enable it and false to disable it.
- concurrentRequest number of concurrent requests to be limited.
- queueSize -1 unlimited queue size, which might use a lot of memory. > 1 integer will limit the requests to be queued, and once the queue is full, 503 will be returned for new requests by default. 
- errorCode can be changed to 429 if it is used internal to throttle the concurrent request to backend APIs. 

By default, the limit.key is server and you can define the limit.server in values.yml like the following. 

```
limit.server:
  /v1/address: 10/s
  /v2/address: 1000/s

```

When limit.key is set as address, you can define the limit.address in values.yml as the following. 

```
limit.address:
  192.168.1.100: 10/h 1000/d
  192.168.1.101: 1000/s 1000000/d
  192.168.1.102:
    /v1/address: 10/s
    /v2/address: 100/s

```

When limit.key is set as client, you can define the limit.client in values.yml as the following.

```
limit.client:
  f7d42348-c647-4efb-a52d-4c5787421e73: 100/m 10000/d
  f7d42348-c647-4efb-a52d-4c5787421e74: 10/m 1000/d
  f7d42348-c647-4efb-a52d-4c5787421e75:
    /v1/address: 10/s
    /v2/address: 100/s

```

Please not that the client_id in the above configuration is from the JWT token and JwtVerifyHandler must be in the chain before the limit handler.

When limit.key is set as user, you can define the limit.user in values.yml as the following. 

```
limit.user:
  alex@lightapi.net: 10/m 10000/d
  alice@lightapi.net: 20/m 20000/d
  albert@lightapi.net:
    /batch: 1000/m 1000000/d

```

Please note that the user_id is from the JWT token and authorizaiton code flow should be used in this case to protect the single page application or mobile application. 

There are several built-in KeyResolver implementation that you can use when client, address or user is selected as the limit.key. Here are several of them that you can choose from. 

```
# Client id Key Resolver.
limit.clientIdKeyResolver: com.networknt.limit.key.JwtClientIdKeyResolver
# Ip Address Key Resolver.
limit.addressKeyResolver: com.networknt.limit.key.RemoteAddressKeyResolver
# User id Key Resolver.
limit.userIdKeyResolver: com.networknt.limit.key.JwtUserIdKeyResolver

```

If the above implementation is not suitable for your business requirement, you can create customized on and replace the above in the configuation. 

