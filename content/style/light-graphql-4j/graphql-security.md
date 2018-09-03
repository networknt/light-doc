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
reviewed: true
---

Graphql Security verifies JWT token in request header and verifies scopes if it is enabled. It is very similar with swagger-security / openapi-security but as there is no swagger specification we cannot verify scopes against the specification. GraphQL recommend authorization 
outside of schema so that we can only verify query scope and mutation scope for read and write access.

This is the handler that should be put before graphql-validator. There is no need to do any validation if JWT token does not exist in the request header.

From release 1.5.18, the light platform supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a security configuration file for different frameworks, a new graphql-security.yml with the same content has been introduced. The security.yml is still loaded if graphql-security.yml doesn't exist for backward compatibility. 

Here is an example of graphql-security.yml

```
# Security configuration for graphql-security in light-graphql-4j. It is a specific config
# for GraphQL framework security. It is introduced to support multiple frameworks in the
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
