---
title: "Graphql Security"
date: 2017-11-29T21:00:02-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Graphql Security verifies JWT token in request header and verifies scopes if it is enabled.
It is very similar with swagger-security / openapi-security but as there is no swagger 
specification we cannot verify scopes against specification. GraphQL recommend authorization 
outside of schema so that we can only verify query scope and mutation scope for read and write 
access.



This is the handler that should be put before graphql-validator. There is no need to do any 
validation if JWT token does not exist in the request header.

The module share the same security.yml and here is an example.

```
# Security configuration in light framework.
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
    '100': oauth/primary.crt
    '101': oauth/secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging for audit. This is to log the entire token
# or choose the next option that only logs client_id, user_id and scope.
logJwtToken: true

# Enable or disable client_id, user_id and scope logging if you don't want to log
# the entire token. Choose this option or the option above.
logClientUserScope: false

# If OAuth2 provider support http2 protocol. If using light-oauth2, set this to true.
oauthHttp2Support: true

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: false
```