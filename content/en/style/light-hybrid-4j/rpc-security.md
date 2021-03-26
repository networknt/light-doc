---
title: "Rpc Security"
date: 2018-05-07T21:13:37-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---
This handler is part of the [light-hybrid-4j][] which is built on top of light-4j but focused only on the hybrid style of API.

It supports OAuth2 with JWT token distributed verification and can be extended to other authentication and authorization approaches. 

### JwtVerifyHandler

This is the handler that is injected during server startup if security.yml enableVerifyJwt is true. It does further scope verification if enableVerifyScope is true against hybrid schema specification.

From release 1.5.18, Light supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a security configuration file for different frameworks, a new hybrid-security.yml with the same content has been introduced. The security.yml is still loaded if hybrid-security.yml doesn’t exist for backward compatibility.

Here is an example of hybrid-security.yml

```
# Security configuration for hybrid-security in light-hybrid-4j. It is a specific config
# for Hybrid framework security. It is introduced to support multiple frameworks in the
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
    '101': secondary.crt
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

Unlike with simple web tokens, the resource server has to contact the Authorization server to validate the bearer token. JWT can be verified by the resource server as long as the token signing certificate is available at the resource server.

[light-hybrid-4j]: /style/light-hybrid-4j/

