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
---


light-4j and related frameworks integrate with light-oauth2 very closely so that tokens
issued by light-oauth2 can be verified on services built on top of light*4j frameworks
independently. We call this centralized security policy management on light-oauth2 and
distributed policy enforcement on services. This eliminate the needs to call oauth2 server 
to verify token for each request to a service. If we do that, then the oauth2 server will 
become a bottleneck as the number of calls between services are significantly higher then 
traditional web services architecture.

In order to do token verification distributed, we have to use PKI to sign the JWT token
and distribute the public key certificate to each running service. Each service will use
the public key certificate to verify token signature to ensure that the token is issued 
by the right OAuth 2.0 provider and the claims are not tempered.

In traditional web service architecture, the public key certificate normally will be copied
to the target host which the services are deployed during first deployment and then copied
the new public key certificate once it is rotated. This is doable because we know the IP
address of the service and we can locate the exact host to deploy the certificate. However,
in pure microservices architecture, all services are in Docker container and orchestrated
by Kubernetes. You don't have the exact address of each service and one service can be down
on one VM and be started on another VM the next seconds. The traditional way to push the
certificate to service is not an option anymore. 

The solution we use is to pull the certificate from light-oauth2 by services whenever 
necessary. 


## Bootstrap certificates

When a service is generated from light-codegen, it will include two public key certificates
for testing. These will be replaced once you move to an official testing environment within
your organization as you services will be using an official testing oauth2 server with a
purchased key pair or different self-signed key pair. When the service goes to production, 
the production certificate will be deployed to the server in externalized config in order to 
bootstrap the service on production. Basically, this will ensure that all services will be 
ready to verify JWT tokens immediately.

## Which key to use

As mentioned above, light-4j supports multiple public key certificates at anytime because
once key is rotated, service still need to use the old key to verify tokens sign by the old
private key as these tokens were issued before the new key is deployed and they are not
expired yet. 

Here is the security.yml config file for security handler.

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

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false

# If OAuth2 provider support http2 protocol. If using light-oauth2, set this to true.
oauthHttp2Support: true

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

```

As you can see the certificate configuration is a map between kid and the key path and filename 
relative to config folder. When light-oauth2 server issues a token, it will put the kid into the 
header of the token. When service receives a token, it will check the kid first and then lookup 
the certificate map to figure out which certificate is needed to verify the token. The certificates 
are loaded during server startup so the lookup and verification should be really fast. 

## Rotate certificate

Once the key changed on the light-oauth2 server, it will be using the new key to sign the
token and the new token will have a new kid for example "102". Once the service receives
the token it checked the local key map and cannot find the key with kid = 102, then it will
call light-oauth2 key service with kid=102 in the path. Once the key is downloaded to the
local, it will be cached for subsequent token verifications. 

Here is the document for [key distribution][] service in light-oauth2.

## Client module

The [client module] provided by light-4j is hiding a lot of interactions with OAuth2 server. For
example, it will renew the access token in a separate thread before token is expired. It downloads
the public key certificate if it doesn't exist on the local config and cache it for future use. 

Here is an example of client.yml and you can see that the endpoints for OAuth2 server defined.

```yaml
# This is the configuration file for Http2Client.
---
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: tls/client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: tls/client.keystore
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: http://localhost:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: 3798d583-275c-47d7-bf46-a3c436846336
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://localhost:8080/authorization_code
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - customer.r
      - customer.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: 6e9d1db3-2feb-4c1f-a5ad-9e93ae8ca59d
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - account.r
      - account.w
  # light-oauth2 key distribution endpoint configuration
  key:
    # key distribution server url
    server_url: http://localhost:6886
    # the unique service id for key distribution service
    serviceId: com.networknt.oauth2-key-1.0.0
    # the path for the key distribution endpoint
    uri: "/oauth2/key"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: 6e9d1db3-2feb-4c1f-a5ad-9e93ae8ca59d

``` 

 
[key distribution]:  /service/oauth/service/key/
[client module]: /concern/client/

