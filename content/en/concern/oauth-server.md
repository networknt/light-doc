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

Most monolithic gateways will have a built-in OAuth 2.0 server to issue tokens to the consumer and verify the tokens for subsequent requests. Consumer applications might cache a token and go to the gateway to get a new token when the cached token is about expired. When migrating to the light-proxy-client, getting a token from the OAuth 2.0 server is taken care of by the light-proxy-client automatically. There is no need to issue a gateway token anymore; however, this getting token from gateway logic might be built-in already. To ensure that there is no code change for the consumer applications, we need to provide a dummy OAuth 2.0 server on the light-proxy-client and light-gateway to return a dummy token for the existing application to complete the invocation flow. 

The current implementation is designed based on the SAG gateway, and only the client_credentials token is supported. However, we are trying to implement it as generic as possible and leave the door open for future enhancements.

### Implementation

The OAuthServerHandler is located in the egress-router module in the light-4j repository. The handler accepts the request body with the following data elements in a JSON format. 

client_id, client_secret and grant_type. The grant_type should always be client_credentials, and the client_id and client_secret should be configured in the oauthServer.yml client_credentials section, separated with a colon. 

You must have a content type header with the value "application/json" header value.

If grant_type is not "client_credentials", then the error response looks like this. 

```
ERR12001:
  statusCode: 400
  code: ERR12001
  message: UNSUPPORTED_GRANT_TYPE
  description: Unsupported grant type %s. Only authorization_code and client_credentials
    are supported.
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

This handler is expecting that the RequestBodyInterceptor is used so that we can get the request body in a map structure in the handler.

When using values.yml to configure the OAuth Server, add the following section in YAML format.

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

Send the following request. 

```
curl -k --location --request POST 'https://localhost:8443/oauth/token' \
--header 'Content-Type: application/json' \
--data-raw '{
    "grant_type": "client_credentials",
    "client_id": "client_id1",
    "client_secret": "client_secret1"
}'
```

And you will have the following response.

```
{
    "access_token": "Nt78M6nEQd-RL0t9Cwiyzw",
    "token_type": "bearer",
    "expires_in": 600
}
```

When using this handler in the light-client-proxy to integrate a legacy consumer application into the ecosystem, we usually allow the consumer to send to the LCP the dummy token returned from the LCP in the Authorization request header. However, we don't want to send this dummy token to the downstream API as they expect a real JWT token from its OAuth 2.0 provider. This means the LCP should enable the TokenHandler to grab the token and replace the dummy token in the Authorization header. 

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


