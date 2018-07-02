---
title: "Provider Registration"
date: 2017-11-10T14:50:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The service to support federated OAuth 2.0 providers.

For example, external OAuth 2.0 provider for external clients and internal OAuth 2.0 provider for internal clients.

The Internal AS needs to have the public key certificate from External AS in order to allow all resource server to verify the tokens signed by both servers.




## Provider workflow


### Provider workflow diagram

![Provider workflow](/images/light-oauth-provider.png)


### Provider workflow steps


Pre-steps, Light-oauth server B and light-oauth server C register as light-oauth security provider on light-Oauth server A. Please follow Provider Registration section for detail of registration.



1.  Client get token from local light-oauth server (from the diagram, Light-Oauth B). The token header includes certificate Key Id


```
   {
     "kid": "05100",
     "alg": "RS256"
   }
```

2.  Client try to access the service which registered on light-oauth A with the access token;


3. Service will check the light-oauth server (Light-Oauth A) for the token to verify the access. The light-oauth server (Light-Oauth A) will try get the local certificate based on the key Id in the token header


4. If light-oauth server (Light-Oauth A) can get certificate directly from local server, verify the token directly. If light-oauth server (Light-Oauth A) cannot get certificate directly from local server, then light-oauth key service will try to get the certificate from registered provider.


5. If light-oauth key service will try to get the certificate from registered provider, then verify the token with the certificate. Otherwise, return with access error.





## Provider Registration

To add a new provider.


```
curl -X POST \
  https://localhost:6889/oauth2/provider \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"providerId":"05","serverUrl":"http://google.ca/light-4j:8080","uri":"/oauth/key","providerName":"cloud light-4j"}'

 ```

To update a provider with new serverUrl.


```
curl -X POST \
  https://localhost:6889/oauth2/provider \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"providerId":"05","serverUrl":"http://google.ca/light-4j:8080","uri":"/oauth/key","providerName":"cloud light-4j"}'

 ```

To delete a provider by the providerId.

```
curl -X DELETE \
  https://localhost:6889/oauth2/provider/02 \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"providerId":"05","serverUrl":"http://networknt/light-4j:8080","uri":"/oauth/key","providerName":"cloud light-4j"}'

 ```



To query all provider.

```
curl -X GET \
  https://localhost:6889/oauth2/provider \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"providerId":"02","serverUrl":"http://yahoo.ca/light-4j:8080","uri":"/oauth/key","providerName":"cloud light-4j"}'

 ```

 Result of the query request:

 ```
{
    "05": {
        "providerId": "05",
        "serverUrl": "http://networknt/light-4j:8080",
        "uri": "/oauth/key",
        "providerName": "cloud light-4j"
    },
    "02": {
        "providerId": "02",
        "serverUrl": "http://yahoo.ca/light-4j:8080",
        "uri": "/oauth/key",
        "providerName": "cloud light-4j"
    }
}

  ```