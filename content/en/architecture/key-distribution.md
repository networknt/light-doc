---
title: "Key Distribution"
date: 2017-11-06T11:14:26-05:00
description: "JWT Public Key Certification Distribution"
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---


The light-4j and related frameworks integrate with light-oauth2 very tightly so that tokens issued by light-oauth2 can be verified on services built on top of light-4j frameworks independently. We call this centralized security policy management on light-oauth2 and distributed policy enforcement on services. It eliminates the need to call oauth2 server to verify a token for each request to a service. If we do that, then the oauth2 server will become a bottleneck as the number of calls between services are significantly higher then traditional web services architecture.

In order to do token verification distributedly, we have to use PKI to sign the JWT token and distribute the public key certificate to each running service. Each service will use the public key certificate to verify token signature to ensure that the token is issued by the right OAuth 2.0 provider and the claims are not tempered.

In traditional web service architecture, the public key certificate normally will be copied to the target host which the services are deployed during initial deployment and then copied the new public key certificate once it is rotated. It is doable because we know the IP address of the service and we can locate the exact host to deploy/copy the certificate. However, in pure microservices architecture, all services are in Docker container and orchestrated by Kubernetes. You don't have the exact address of each service, and one service can be down on one VM and be started on another VM the next seconds. The traditional way to push the certificate to the service is not an option anymore.  

The solution we use is to pull the certificate from light-oauth2 by services whenever necessary. 

## Bootstrap certificates

When a service is generated from the light-codegen, it will include two public key certificates for testing. These will be replaced once you move to an official testing environment within your organization as your services will be using an official testing oauth2 server with a purchased key pair or different self-signed key pair. When the service goes to production, the production certificate will be deployed to the server in externalized config in order to bootstrap the service on production. Basically, this will ensure that all services will be ready to verify JWT tokens immediately.

## Which key to use

As mentioned above, light-4j supports multiple public key certificates at any time because once a key is rotated, the service still needs to use the old key to verify tokens signed by the old private key as these tokens were issued before the new key is deployed and they are not expired yet.

Here is the security.yml config file for security handler.

```yaml
# Security configuration for security module in light-4j. For each individual framework,
# it has a framework specific security config file to control if security and scopes
# verification are enabled or not.

# This configuration file is only for JwtHelper class most of the cases. However, if there
# is no framework specific security configuration available. The fallback security config
# is read from this file. Hence we leave the enableVerifyJwt and enableVerifyScope to true.
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

# Enable or disable JWT token logging for audit. This is to log the entire token
# or choose the next option that only logs client_id, user_id and scope.
logJwtToken: true

# Enable or disable client_id, user_id and scope logging if you don't want to log
# the entire token. Choose this option or the option above.
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

As you can see the certificate configuration is a map between a kid and a key filename. When light-oauth2 server issues a token, it will put the kid into the header of the token. When service receives a token, it will check the kid first and then look up the certificate map to figure out which certificate is needed to verify the token. The certificates are loaded during server startup so the lookup and verification should be really fast. 

## Rotate certificate

Once the key changed on the light-oauth2 server, it will be using the new key to sign the token, and the new token will have a new kid for example "102". Once the service receives the token it checked the local keymap and cannot find the key with kid = 102; then it will call light-oauth2 key service with kid=102 in the path. Once the key is downloaded to the local, it will be cached for subsequent token verifications. 

Here is the document for [key distribution][] service in light-oauth2.

## Client module

The [client module] provided by light-4j is hiding a lot of interactions with OAuth2 server. For example, it will renew the access token in a separate thread before a token is expired. It downloads the public key certificate if it doesn't exist on the local config and cache it for future use. 

Here is the built-in [client.yml](https://github.com/networknt/light-4j/blob/master/client/src/main/resources/config/client.yml) and you can see that the endpoints for OAuth2 server defined.

 
[key distribution]:  /service/oauth/service/key/
[client module]: /concern/client/
