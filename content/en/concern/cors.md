---
title: "CORS"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

If your API server serves a SPA (single page application) built on top of Angular or React, there is no issue when the SPA accesses APIs on the same server. However, some of the single page applications are served by another server on another domain. In this case, the API server has to handle the pre-flight options request in order to allow browser clients to access the APIs directly. Also, it has to handle the subsequent requests with the Origin header to match it with the cors.yml configuration.


## CorsHttpHandler

This handler handles the HTTP pre-flight option request and returns the correct header to the client. The information returns to the client is controlled by cors.yml configuration file.

## Configuration

Here is the default configuration cors.yml

```
---
# If cors handler is enabled or not
enabled: ${cors.enabled:true}

# Allowed origins, you can have multiple and with port if port is not 80 or 443. This is the global
# configuration for all paths. If you want to have different configuration for different paths, you
# can use pathPrefixAllowed. The value is a list of strings.
# Wildcard is not supported for security reasons.
allowedOrigins: ${cors.allowedOrigins:}
#  - http://localhost

# Allowed methods list. The value is a list of strings. The possible value is GET, POST, PUT, DELETE, PATCH
# This is the global configuration for all paths. If you want to have different configuration for different
# paths, you can use pathPrefixAllowed.
allowedMethods: ${cors.allowedMethods:}
#  - GET
#  - POST

# cors configuration per path prefix on a shared gateway. You either have allowedOrigins and allowedMethods
# or you have pathPrefixAllowed. You can't have both. If you have both, pathPrefixAllowed will be used.
# The value is a map with the key as the path prefix and the value is another map with allowedOrigins and
# allowedMethods. The allowedOrigins is a list of strings and allowedMethods is a list of strings.

# Use the above global configuration if you are dealing with a single API in the case of http-sidecar,
# proxy server or build the API with light-4j frameworks. If you are using light-gateway with multiple
# downstream APIs, you can use the pathPrefixAllowed to set up different CORS configuration for different
# APIs.


# Here is an example in values.yml
# cors.pathPrefixAllowed:
#   /v1/pets:
#     allowedOrigins:
#       - https://abc.com
#       - https://www.xyz.com
#     allowedMethods:
#       - GET
#       - PUT
#       - POST
#       - DELETE
#   /v1/market:
#     allowedOrigins:
#       - https://def.com
#       - https://abc.com
#     allowedMethods:
#       - GET
#       - POST
pathPrefixAllowed: ${cors.pathPrefixAllowed:}

```

## 