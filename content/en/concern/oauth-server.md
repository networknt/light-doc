---
title: "OAuth Server"
date: 2022-11-04T19:59:17-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When an organization migrates existing applications and services to the light-gateway from a monolithic gateway without changing the code for all consumer applications, we need to provide a dummy OAuth 2.0 server on the light-gateway to simulate how other gateway works. 

Most monolithic gateways will have a built-in OAuth 2.0 server to issue tokens to the consumer and verify the tokens for subsequent requests. Consumer applications might cache a token and go to the gateway to get a new token when the cached token is about expired. When migrating to the light-proxy-client, getting a token from the OAuth 2.0 server is taken care of by the light-proxy-client automatically. There is no need to issue a gateway token anymore; however, this getting token from gateway logic might be built-in already. To ensure that there is no code change for the consumer applications, we need to provide a dummy OAuth 2.0 server on the light-proxy-client and light-gateway to return a dummy token or a real token from an OAuth 2.0 provider for the existing application to complete the invocation flow. 

The current implementation is designed based on the SAG gateway, and only the client_credentials token is supported. However, we are trying to implement it as generic as possible and leave the door open for future enhancements.

### Implementation

The OAuthServerHandler is located in the egress-router module in the light-4j repository. The handler accepts the request body with the following data elements in a JSON format or FormData. 

client_id, client_secret and grant_type. The grant_type should always be client_credentials, and the client_id and client_secret should be configured in the oauthServer.yml client_credentials section, separated with a colon. 

You must have a content type header with the value "application/json", "multipart/form-data" or "application/x-www-form-urlencoded".

If content type is missing, you will receive the following error. 

```
ERR10076:
  statusCode: 400
  code: ERR10076
  message: CONTENT_TYPE_MISSING
  description: Content type header is missing and it is required.
```

If the content type is not one of the above allowed content types, you will receive the following error. 

```
ERR10077:
  statusCode: 400
  code: ERR10077
  message: INVALID_CONTENT_TYPE
  description: Invalid content type %s.

```


If grant_type is not "client_credentials", then the error response looks like this. 

```
ERR12001:
  statusCode: 400
  code: ERR12001
  message: UNSUPPORTED_GRANT_TYPE
  description: Unsupported grant type %s. Only authorization_code and client_credentials
    are supported.
```

The server will try to get the client_id and client_secret from the request body, if they are not in the body, it will try to get from the Authroization header as a basic credential. 

If the server cannot get client_id and client_secret from the request body and authorization header, then you will receive the following error. 

```
ERR12002:
  statusCode: 401
  code: ERR12002
  message: MISSING_AUTHORIZATION_HEADER
  description: Missing authorization header. client credentials must be passed in
    as Authorization header.

```

If the authorization header does exist, but it is not a basic header, then you will receive the following error. 

```
ERR12003:
  statusCode: 401
  code: ERR12003
  message: INVALID_AUTHORIZATION_HEADER
  description: Invalid authorization header %s.

```

If client_id or client_secret is not matched, then the error response looks like this. 

```
ERR12004:
  statusCode: 401
  code: ERR12004
  message: INVALID_BASIC_CREDENTIALS
  description: Invalid Basic credentials %s.
```

If grant_type, client_id and client_secret are matched, then we get a JSON response that looks like this. 

```
{
  "access_token": "8ccf3dfd-c683-4c36-9f8c-8709e4b57d91",
  "token_type": "bearer",
  "expires_in": 600
}
```

### Configuration

Here is the default configuration in the module. 

oauthServer.yml
```
# oauthServer.yml
# indicate if the handler is enabled or not in the handler chain.
enabled: ${oauthServer.enabled:true}
# A list of client_id and client_secret concat with a colon.
client_credentials: ${oauthServer.client_credentials:}
# An indicator to for path through to an OAuth 2.0 server to get a real token.
passThrough: ${oauthServer.passThrough:false}
# If pathThrough is set to true, this is the serviceId that is used in the client.yml configuration as the key
# to get all the properties to connect to the target OAuth 2.0 provider to get client_credentials access token.
# The client.yml must be set to true for multipleAuthServers and the token will be verified on the same LPC.
tokenServiceId: ${oauthServer.tokenServiceId:light-proxy-client}

```

This handler is expecting that the RequestBodyInterceptor is used so that we can get the request body in a map structure in the handler.

When using values.yml to configure the OAuth Server, add the following section in YAML format to use a dummy token without verification.

```
# oauthServer.yml
oauthServer.client_credentials:
  - client_id1:client_secret1
  - client_id2:client_secret2
  - 5f6906ef-d2f7-4f65-8c04-25e054b352fa:102bd89c-d946-4713-a045-80d9c425d82f

```

Or add the following section in JSON format if the config server is used.

```
# oauthServer.yml
oauthServer.client_credentials: ["client_id1:client_secret1","client_id2:client_secret2","5f6906ef-d2f7-4f65-8c04-25e054b352fa:102bd89c-d946-4713-a045-80d9c425d82f"]

```

Or add the following section in a comma-separated string if the config server is used.

```
# oauthServer.yml
oauthServer.client_credentials: client_id1:client_secret1,client_id2:client_secret2,5f6906ef-d2f7-4f65-8c04-25e054b352fa:102bd89c-d946-4713-a045-80d9c425d82f

```

If you want to have a real token to be retrieved from an OAuth 2.0 provider, you need to change the passThrough flag to true and provide a serviceId as the key for the client credentials token retrieval configuration in the client.yml section. If you don't want to name the serviceId, you can use the light-proxy-client as the serviceId by default.


```
# oauthServer.yml
oauthServer.client_credentials:
  - client_id1:client_secret1
  - client_id2:client_secret2
  - 5f6906ef-d2f7-4f65-8c04-25e054b352fa:102bd89c-d946-4713-a045-80d9c425d82f
oauthServer.passThrough: true

# client.yml
client.tokenCcServiceIdAuthServers:
  light-proxy-client:
    server_url: https://networknt.oktapreview.com
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: false
    # token endpoint for client credentials grant for LPC
    uri: /oauth2/aus9xt6ef1cSYyRPH1d6/v1/token
    # client_id for client credentials grant flow for LPC
    client_id: 0oa3fbpejuouYqXRu1d7
    client_secret: NPvsFwaEQEz1mnfKX_89LY8l_aLBbJjD3wObeZKx
    scope:
      - petstore.write

```


### Usage

This handler should only be used in the light-gateway to issue dummy tokens.

In the handler.yml file in src/resources/config folder, we have the following endpoint defined. 

```
  - path: '/oauth/token'
    method: 'post'
    exec:
      - oauth
```

For organizations that use customized light-gateway configuration, you can change the endpoint path to match your current monolithic gateway path. 

In the values.yml file we need to register the handler and give it an alias as oauth. 

```
handler.handlers:
  .
  .
  .
  - com.networknt.router.OAuthServerHandler@oauth

```

Add a section for oauthServer.yml in the values.yml file.

```
# oauthServer.yml
oauthServer.client_credentials: ["client_id1:client_secret1","client_id2:client_secret2","5f6906ef-d2f7-4f65-8c04-25e054b352fa:102bd89c-d946-4713-a045-80d9c425d82f"]

```

As the api-key is in a new module, we need to make sure that the light-gateway pom.xml has this module as dependency. 

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>api-key</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

After completing the above configuration, let's start the light-gateway locally to have a test.

Send the following request with client_id and client_secret in the request body with JSON format. 

```
curl -k --location --request POST 'https://localhost:8443/oauth/token' \
--header 'Content-Type: application/json' \
--data-raw '{
    "grant_type": "client_credentials",
    "client_id": "client_id1",
    "client_secret": "client_secret1"
}'
```

Send the following request with client_id and client_secret in the BASIC authorization header with x-www-form-urlencoded format. 

```
curl -k --location --request POST 'https://localhost:8443/oauth/token' \
--header 'Authorization: BASIC client_id1:client_secret1' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials'
```

Send the following request with client_id and client_secret based 64 encoded in the BASIC authroization header with x-www-form-urlencoded format. 

```
curl --location --request POST 'https://localhost:8443/oauth/token' \
--header 'Authorization: BASIC Y2xpZW50X2lkMTpjbGllbnRfc2VjcmV0MQ==' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials'
```

And you will have the following response.

```
{
    "access_token": "Nt78M6nEQd-RL0t9Cwiyzw",
    "token_type": "bearer",
    "expires_in": 600
}
```

Or the following response if passThrough is true. 

```
{
    "access_token": "eyJraWQiOiJHSlYyUThRUUsxTkF1ZlJ1WHFORVpmc2R3el85MUVRQmR0Q284Snk4RS1RIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULm1lWUNxWjR0RFlnTnBmcWZTb25fTHlOTEtGR1hfdEpVT0FUMXJ0RHZDOWMiLCJpc3MiOiJodHRwczovL3N1bmxpZmVhcGkub2t0YXByZXZpZXcuY29tL29hdXRoMi9hdXM5eHQ2ZGQxY1NZeVJQSDFkNiIsImF1ZCI6ImRldi5jYW5hZGEuSW5kLnN1bmxpZmVhcGkub2t0YXByZXZpZXcuY29tIiwiaWF0IjoxNjY5NDEyMTM4LCJleHAiOjE2Njk0MTU3MzgsImNpZCI6IjBvYTNmYnBlanVvdVlxWFJ1MWQ3Iiwic2NwIjpbImluZC5tcmFzZXZlbnRtYXBwZXIud3JpdGUiXSwic3ViIjoiMG9hM2ZicGVqdW91WXFYUnUxZDcifQ.rsYeyuvxaLxOAFMAGhW_Ai88xUs_GchH94TkO2ybNGM8J41YWXIaxLnORv_JkM02AIA82d4csjBKK-CN4QpaxWMMOzTwls1FC9hL_jVNti5T69dxpCNzhYBetr-7DlISqh-jRKMo_cX0lyGbzl1g7DxK22wJxrr1Vs_I7Jt8mDTqvpWFf1apDm6Fr1cV2l51_sirzya-jdLo8S_Sm-eHKBH2NiT47R1YJW0JtL0KZxfwl0P0BM6zK4DEgG7P_jjagrwcrbsw0QSj8iLLdjdO1WSccXjqn11ecYNo75M2ssxL6f6_nM0EWuCl-1a1493u3jOpfKMXKccYXs_xT4LABC",
    "token_type": "bearer",
    "expires_in": 3594
}
```

When using this handler in the light-client-proxy to integrate a legacy consumer application into the ecosystem, we usually allow the consumer to send to the LCP the dummy token or real token returned from the LCP in the Authorization request header. 

Suppose there is only one consumer application on the host; you can choose the dummy token as there are no security concerns between the consumer application and the LPC, and the LPC is dedicated to the consumer application. 

Suppose there are several consumer applications on the same host. We need to ensure that only the authorized consumer applications can connect to the LPC to send a request to access downstream APIs. In that case, we can set the passThrough to true and allow the LPC to get a real token from an OAuth 2.0 provider. 

When a dummy token is used, we don't want to send it to the downstream API as they expect a real JWT token from its OAuth 2.0 provider. This means the LCP should enable the TokenHandler to grab the token and replace the dummy token in the Authorization header.

As the light-4j application supports two tokens, with primary in the Authorization header and secondary in the X-Scope-Token header, we have to remove the dummy token in order to allow the TokenHandler to put the real token into the same Authorization header. Otherwise, the LCP will put the real token into the X-Scope-Token header because the dummy token occupies the Authorization header. 

To remove the Authorization header, we can config the header.yml section in the values.yml file. 

For example.

```
# header.yml
header.enabled: true
header.pathPrefixHeader:
  #Flinks Fin banking
  /v3/BankingServices/GetNightlyRefreshStatus:
    request:
      remove:
        - Authorization
```

This will allow the HeaderHandler to remove the Authorization header if the request path is "/v3/BankingServices/GetNightlyRefreshStatus". 

When a real token is used, we can send it to the downstream API in the Authorization header; however, we also need to get another token that contains the right scope to access the downstream API and put it in the X-Scope-Token header. LPC will verify the token in the Authorization header, and the downstream API will verify both to allow the request to access. 

Also, you need to setup the client.yml, token.yml sections to get an access token for the request. 

```
# token.yml
token.enabled: true
token.appliedPathPrefixes:
  - /v3/BankingServices/GetNightlyRefreshStatus
```

```
client.pathPrefixServices:
  /v3/BankingServices/GetNightlyRefreshStatus: BankingServices

client.tokenCcServiceIdAuthServers:
  BankingServices:
    server_url: https://networknt.oktapreview.com
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: false
    # token endpoint for client credentials grant for Mras
    uri: /oauth2/aus9xt6dd1cSYyRPG2d6/v1/token
    # client_id for client credentials grant flow for Mras.
    client_id: 0oa3fbpej3ou2qXRu1d7
    client_secret: NPxfFwaC903mnfKX_29LY8l_aLBbJjD3wObeZKx
    scope:
      - api.write
```

Note: The above OAuth credentials are fake and you need to replace them with your own server_url, uri, client_id, client_secret and scope. 


