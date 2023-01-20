---
title: "Okta Oauth 2.0 Provider"
date: 2023-01-19T18:49:47-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We have too many configuration examples to leverage the [light-oauth2][] or the [kafka-oauth][] for security. In this tutorial, we are going to show you how to use the Okta cloud OAuth 2.0 provider for security. One of our customers is using Okta cloud for hundreds of services. 


### Single Okta JWK

This is the setup for a light-4j service or http-sidecar to verify the incoming JWT token by downloading the JWK from the same OAuth server instance on the cloud. I am assuming you are using REST API with light-rest-4j framework. 

To use the JwtVerifyHandler, we need to register it in the handlers section in handler.yml and also put the alias into the default chain. 

handler.yml

```
handlers:
  .
  .
  .	
  - com.networknt.openapi.JwtVerifyHandler@security
  .
  .
  .

chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation 
    - specification
    - security
    .
    .
    .
```

In the openapi-security.yml section in values.yml, we need to enable the security to verify the JWT token and also the scopes in the token against the specification. To load the JWK during the server startup, we also, need to set up the bootstrap from JWK server to true. We also need to change the keyResolver to JsonWebKeySet. 

In the client.yml section, we need to define the tokenKeyServerUrl and tokenKeyUri so that the server knows where to load the JWK. 

values.yml
```
# openapi-security.yml
openapi-security.enableVerifyJwt: true
security.enableVerifyScope: true
security.keyResolver: JsonWebKeySet
security.bootstrapFromKeyService: true

# client.yml
client.tokenKeyServerUrl: https://networknt.oktapreview.com
client.tokenKeyUri: /oauth2/aus133d83gC9EXGoS1d7/v1/keys

```

With the above configuration, you can get a JWT token from Okta from the token endpoint https://networknt.oktapreview.com//oauth2/aus133d83gC9EXGoS1d7/v1/token and put it into the request Authorization header. The request will be verified by the server with the JWK downloaded from the same Okta instance. 

### Automatically Retrieve JWT Token

This is normally used in the http-sidecar or light-proxy-client to retrieve a JWT token and injected into the outgoing request. 


### Multiple OAuth Providers

This is used in a light-gateway instance to verify JWT tokens from multiple JWK instances and retrieve JWT from multiple JWT token servers. 


[light-oauth2]: /service/oauth/
[kafka-oauth]: /service/oauth-kafka/
