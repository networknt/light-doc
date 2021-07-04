---
title: "Swagger Security"
date: 2017-11-22T22:45:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This handler is part of the [light-rest-4j][] which is built on top of light-4j but 
focused on RESTful API only. Also, it only works with Swagger 2.0 specification.

It supports OAuth2 with JWT token distributed verification and can be extended to 
other authentication and authorization approaches. 

### JwtVerifyHandler

This is the handler that is injected during server start up if security.yml 
enableVerifyJwt is true. It does further scope verification if enableVerifyScope 
is true against swagger specification.

From release 1.5.18, Light supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a security configuration file for different frameworks, a new swagger-security.yml with the same content has been introduced. The security.yml is still loaded if swagger-security.yml doesn't exist for backward compatibility. 

Here is an example of swagger-security.yml

```
# Security configuration for swagger-security in light-rest-4j. It is a specific config
# for Swagger framework security. It is introduced to support multiple frameworks in the
# same server instance. If this file cannot be found, the generic security.yml will be
# loaded for backward compatibility.
---
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: true

# User for test only. should be always be false on official environment.
enableMockJwt: false

# JWT signature public certificates. kid and certificate path mappings.
jwt:
  certificate:
    '100': primary.crt
    '101': tlsecondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: false

```

### Distributed JWT verification

Unlike simple web tokens, the resource server has to contact Authorization server 
to validate the bearer token. JWT can be verified by resource server as long as 
the token signing certificate is available at resource server. 


[light-rest-4j]: https://github.com/networknt/light-rest-4j


