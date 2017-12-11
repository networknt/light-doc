---
title: "Security"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 93
weight: 93	#rem
aliases: []
toc: false
draft: false
---

API security is very important and today most APIs don't have security built-in at all but
relying on third party API gateway to handle the security at network level. This assumes
that within the organization, it is safe or some sort of firewall setup to ensure that
only Gateway host can access the service host. 

Once we move to the cloud, everything is dynamic and firewall won't work anymore as services
can be down on one VM but started in another VM immediately with container orchestration tools.

The traditional gateway also add another layer of network calls and if there are so many 
service to service calls, the added up latency is unacceptable. 

Here is an [gateway article][] in the architecture section that talks about the drawbacks of the API 
gateway and why it is not suitable for microservices architecture. 

In order to make sure that security works for microservies, we have extended the OAuth 2.0
specification and come up with a brand new approach for authorization with JWT. Here is an
[security article][] talks about the API security in light*4j frameworks. 

Basically, we have [light-oauth2][] OAuth 2.0 provider and built-in JWT security verification 
in the frameworks. The security policy is managed by the OAuth2 provider with service registration 
and client registration; however, the policy enforcement is done distributed at each service level. 
With PIK signed JWT token issued from the light-oauth2, all services can verify the token with the 
JWT signature public key certificate. 

The following is a list of key components in light-4j security.

## light-oauth2 server

Light-4j and related frameworks support OAuth 2.0 and related specifications to authorize service 
access. By default, the framework contains two pairs of public key certificates issued by our own 
testing oauth2 server which can be installed from Docker. For more info, please refer to 
https://github.com/networknt/light-oauth2

Light-4j also provides a client module that can communicate with light-oauth2 in the background to
get access token and renew the access token before it is expired. 

## Kid

Since services are deployed in the cloud without static IP, the traditional push certificates to each 
service is not working anymore. In this framework each service will pull certificate from OAuth2 server 
key service by a kid from JWT token header if the key doesn't exist locally. This approach that uses
kid to identify public key certificate to verify the JWT token also helps if you have more than one key
used on your OAuth 2.0 provider. One use case is that you must support at least two keys during key
rotation on a yearly basis due to maximize the security. 


## JwtMockHandler

This is a testing OAuth2 endpoints provider and it can be injected into the handler chain for unit 
testing so that it won't depend on an instance of light-oauth2 for unit tests. It can be used by other
frameworks and also services developed on top of these frameworks. 

## Long lived token

To make integration test easier, a long lived token is provided by the oauth2 
server and it can be found at https://github.com/networknt/light-oauth2 README.md

## Configuration

Here is the default security.yml

```yaml
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

### Enable security

To enable security, just update enableVerifyJwt to true. If you want to verify the scopes defined in
swagger.json against the scope registered on OAuth 2.0 provider, then set enableVerifyScope to true. 
It is highly recommended to have both enabled on production.

### Enable mock JWT generation

If you want to test with different payloads in JWT token or other parameters with JWT token generation,
then you can set enableMockJwt to true; however, you'd better to create a new security.yml and put it
into src/test/resources/config folder so that it works only with your test cases. 


### Public key certificates

The light-4j framework supports multiple public key certificates and they are defined in the jwt section
with kid that mapped to the certificate file in oauth sub folder from config folder. The light-codegen
generates two public key certificates to bootstrap the service startup and you should replace them with
your own certificate for production usage.

### Clock Skew in seconds

This is part of the jwt configuration and default is 60 seconds. It means if the JWT token receives are expired
already but still within 60 seconds from expires time, then it still consider valid as computer clocks might
not synced perfectly and there are network and prcessing latency as well.

### logJwtToken and logClientUserScope

The JWT token is considered sensitive so the customer needs make decision on how the information will be
logged. The logJwtToken will just log the raw token itself and logClientUserScope will log the three most
important piece of info only. Normally, you choose one of them or disable both. It doesn't make sense to
log both as information is duplicated. 

### oauthHttp2Support

To indicate if the OAuth 2.0 provide support HTTP 2.0 protocol. The default is true as we are assuming
light-oauth2 is used. 

### enableJwtCache

Cache the JWT token within the period of expiry so that we can only check if the token is not expired
without verify the signature as it is very time consuming and CPU intensive. Default is true and you
should only turn it off if you have a very good reason. For example, token chaining is used and every
request has a unique token so cache doesn't make sense in the case. 

### bootstrapFromKeyService

The default is false as we assume that you are in dev without OAuth 2.0 server access or you are using
other OAuth 2.0 server instead of [light-oauth2][]. If the value is false, the server will load public
key certificates for JWT signature verification from the location specified by jwt:certificate in this
config file which are 100 -> oauth/primary.crt and 101 -> oauth/secondary.crt.

If this value is set true, then the public key certificate will be loaded from [light-oauth2 key service][]
dynamically. The key will be loaded once the first JWT token is received, the security middleware handler
will check the kid in the token header and then check if there is cached copy of the public key certificate
map to this kid. If no cache, then send a request to light-oauth2 key service to get the public key cert
with kid as query parameter. To ensure that you are connecting to the right Key service, TLS connection
is a must. In addition, the service itself must be registered on light-oauth2 client service as a client
as well and client_id and client_secret need to be passed in the Authorization header for authentication.
This is to further ensure that the OAuth 2.0 provider is the right one to issue the public key certficates.

This feature will impact the Server Info endpoint provided by all services built on top of light-4j as
fingerprints of OAuth 2.0 public key certificates might not be available locally. The Server Info might
return nothing about this list of fingerprints if this config parameter is set to true. Also the certify
process on light-portal will let it go if no fingerprints returned from Server Info endpoint. That means
the certificates are from light-oauth2 server dynamically and there is no risk the testing certificates
will be packaged into production configuration.    

[gateway article]: /architecture/gateway/
[security article]: /architecture/security/
[light-oauth2]: /service/oauth/
[light-oauth2 key service]: /service/oauth/service/key/
