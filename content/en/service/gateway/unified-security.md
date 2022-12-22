---
title: "Unified Security"
date: 2022-12-21T19:16:46-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


### Introduction

This all-in-one security handler combines Basic, OAuth and API Key to be used in a shared light-gateway instance to handle multiple consumers and providers. In a shard gateway use case, we might need to use the following handlers simultaneously in the request/response chain. 

* [JwtVerifyHandler][]
* [ApiKeyHandler][]
* [BasicAuthHandler][]

This requires the user to position these handlers in the chain carefully so that one handler can skip the verification and pass it to the next handler to do the verification. To simplify the configuration and eliminate confusion, we have created a unified handler to invoke Basic, OAuth and ApiKey handlers based on the unified-security.yml configuration. 

### Configuration

Here is the built-in configuration in the module. 

```
# unified-security.yml
# indicate if this handler is enabled. By default, it will be enabled if it is injected into the
# request/response chain in the handler.yml configuration.
enabled: ${unified-security.enabled:true}
# Anonymous prefixes configuration. A list of request path prefixes. The anonymous prefixes will be checked
# first, and if any path is matched, all other security checks will be bypassed, and the request goes to
# the next handler in the chain. You can use json array or string separated by comma or YAML format.
anonymousPrefixes: ${unified-security.anonymousPrefixes:}
# String format with comma separator
# /v1/pets,/v1/cats,/v1/dogs
# JSON format as a string
# ["/v1/pets", "/v1/dogs", "/v1/cats"]
# YAML format
#   - /v1/pets
#   - /v1/dogs
#   - /v1/cats
pathPrefixAuths: ${unified-security.pathPrefixAuths:}
# format as a string for config server.
# [{"prefix":"/salesforce","basic":true,"jwt":true,"apikey":true,"jwkServiceIds":"com.networknt.petstore-1.0.0, com.networknt.market-1.0.0"},{"prefix":"/blackrock","basic":true,"jwt":true,"jwkServiceIds":["com.networknt.petstore-1.0.0","com.networknt.market-1.0.0"]}]
# format with YAML for readability
    # path prefix security configuration.
    # - pathPrefix: /salesforce
        # indicate if the basic auth is enabled for this path prefix
      # basic: true
        # indicate if the jwt token verification is enabled for this path prefix
      # jwt: true
        # indicate if the apikey is enabled for this path prefix
      # apikey: true
        # if jwt is true and there are two or more jwk servers for the path prefix, then list all the
        # serviceIds for jwk in client.yml
      # jwkServiceIds: service1,service2

```

The above configuration file has some examples of setting up unified-security.anonymousPrefixes and unified-security.pathPrefixAuths. 

The following is an example of values.yml

```
unified-security.anonymousPrefixes:
  - /v1/dogs
  - /v1/cats

unified-security.pathPrefixAuths:
  - prefix: /v1/salesforce
    basic: true
    jwt: true
    apikey: true
    jwkServiceIds: com.networknt.petstore-1.0.0, com.networknt.market-1.0.0
  - prefix: /v1/blackrock
    basic: true
    jwt: true
    jwkServiceIds: ["com.networknt.petstore-1.0.0", "com.networknt.market-1.0.0"]
  - prefix: /v1/test1
    apikey: true
  - prefix: /v1/pets
    jwt: true
```

We need to set up the value in a JSON string when using the config server. 

```
unified-security.anonymousPrefixes: /v1/pets,/v1/cats
unified-security.pathPrefixAuths: [{"prefix":"/salesforce","basic":true,"jwt":true,"apikey":true,"jwkServiceIds":"com.networknt.petstore-1.0.0, com.networknt.market-1.0.0"},{"prefix":"/blackrock","basic":true,"jwt":true,"jwkServiceIds":["com.networknt.petstore-1.0.0","com.networknt.market-1.0.0"]}]

```

To allow the UnifiedSecurityHandler to call other security handlers, we need to register these handlers in the handler.yml and also put the UnifiedSecurityHandler in the default chain. 

Here is the section for handler.yml in the values.yml

```
handler.handlers:
  - com.networknt.openapi.JwtVerifyHandler@jwt
  - com.networknt.apikey.ApiKeyHandler@apikey
  - com.networknt.basicauth.BasicAuthHandler@basic
  - com.networknt.openapi.UnifiedSecurityHandler@security


hnadler.chains.default:
  .
  .
  .
  - security
  .
  .
  .

```

The UnifiedSecurityHandler uses the @jwt, @basic and @apikey to find other handlers and invoke them according to the configuration. So please make sure that you don't change the alias names. 

In the default chain, we only need to put the UnifiedSecurityHandler alias security in the proper position. 

### Basic

In the values.yml file basic-auth.yml section, we should remove the specifial user anonymous and move all the prefixes to the unified-security.anonymousPrefixes section. 

Another specifial user bearer contains prefixes that we want to skip the basic and allow the JwtVerifyHandler to verified the Authorization header can be removed and config all the prefixes in the unified-security.pathPrefixAuths to have the jwt: true. 


### Jwt

If we want to by pass the Jwt verification with the openapi-security.skipPathPrefixes, we can remove it and put these prefixes in the unified-security.anonymousPrefixes section. 

### ApiKey

There is no changes for the apikey.yml configuration in values.yml



[JwtVerifyHandler]: /style/light-rest-4j/openapi-security/
[ApiKeyHandler]: /concern/api-key/
[BasicAuthHandler]: /concern/basic/
