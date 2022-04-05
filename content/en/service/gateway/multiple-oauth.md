---
title: "Multiple OAuth 2.0 Providers"
date: 2022-04-04T22:46:44-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

### Use Cases

There are three deployments that need to support multiple OAuth 2.0 providers: 

* When the light gateway is used as a client gateway or client proxy to bring the legacy client/application to the light ecosystem. The client gateway will be responsible for getting the JWT token on behalf of the native client application. The token needs to be cached and auto-renewed before expiration. 

Suppose several client applications are deployed on the same host and share the same client gateway deployed on the host. In that case, the client gateway will connect to multiple OAuth 2.0 providers to get JWT tokens. 

There are two different situations: 

- All clients access a set of services managed by one OAuth 2.0 server. In this case, the client gateway only needs to go to the same OAuth token URL to get tokens with different client_id/client_secret for each client.

- Multiple OAuth 2.0 servers manage the services, and the client gateway must decide which OAuth server to go to get the JWT token based on the request 
path. 

* When the light gateway is used as the server gateway or server proxy to bring the legacy service/API to the light ecosystem. The server gateway will be responsible for verifying the JWT token by connecting to the OAuth 2.0 provider JWK endpoint to download the JWK. Also, when the service/API calls other services/APIs, it will be responsible for getting the JWT token. 

- On the token verification side, only one OAuth 2.0 provider should be available if there is only one backend service behind the server gateway. However, when multiple services are behind one server gateway on the same host, we need to support multiple OAuth 2.0 servers to get JWK. 

- When the service calls other services/APIs, we might need to connect to multiple OAuth 2.0 providers to get JWT tokens. 

* When the light gateway is used as a standalone gateway server, multiple clients/applications might access multiple services/APIs. This requires the gateway can verify the tokens from multiple JWK URLs if JWT verification is enabled on the gateway. 



### Configuration

The configuration is set up in the client.yml, and all the properties are externalized so that we can use values.yml to overwrite these properties. To have the multiple OAuth 2.0 providers support, we first need to enable it.

```
  # OAuth 2.0 token endpoint configuration
  # If there are multiple oauth providers per serviceId, then we need to update this flag to true. In order to derive the serviceId from the
  # path prefix, we need to set up the pathPrefixServices below if there is no duplicated paths between services.
  multipleAuthServers: ${client.multipleAuthServers:false}
```

The default value is false, and we can change it in the values.yml with the following. 


```
client.multipleAuthServers: true
```

Regardless of getting tokens or jwks, we first need to figure out the serviceId to find the mapped OAuth 2.0 server to get the token or jwk. 

In the client.yml, we have a section that define this mapping. 

```
# If you have multiple OAuth 2.0 providers and use path prefix to decide which OAuth 2.0 server
# to get the token or JWK. If two or more services have the same path, you must use serviceId in
# the request header to use the serviceId to find the OAuth 2.0 provider configuration.
pathPrefixServices: ${client.pathPrefixServices:}
```

Here is an example in the values.yml


```
client.pathPrefixServices:
  /api/petstore: com.networknt.petstore-3.0.1
  /api/market: com.networknt.market-1.0.0
```

Now, we have identify the serviceId with the request path, we can set up the token config with the following. 

```
# The serviceId to the service specific OAuth 2.0 configuration. Used only when multipleOAuthServer is
# set as true. For detailed config options, please see the values.yml in the client module test.
serviceIdAuthServers: ${client.tokenCcServiceIdAuthServers:}

```

Here is an example in the values.yml

```
client.tokenCcServiceIdAuthServers:
  com.networknt.petstore-3.0.1:
    # token server url. The default port number for token service is 6882. If this is set,
    # it will take high priority than serviceId for the direct connection
    server_url: https://localhost:7771
    # token service unique id for OAuth 2.0 provider. If server_url is not set above,
    # a service discovery action will be taken to find an instance of token service.
    # serviceId: ${client.tokenServiceId:com.networknt.oauth2-token-1.0.0}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.tokenProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.tokenProxyPort:3128}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # token endpoint for client credentials grant
    uri: /oauth2/token
    # client_id for client credentials grant flow.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client_secret for client credentials grant flow.
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
    # optional scope, default scope in the client registration will be used if not defined.
    # If there are scopes specified here, they will be verified against the registered scopes.
    scope:
    - petstore.r
    - petstore.w
  com.networknt.market-1.0.0:
    # token server url. The default port number for token service is 6882. If this is set,
    # it will take high priority than serviceId for the direct connection
    server_url: https://localhost:7772
    # token service unique id for OAuth 2.0 provider. If server_url is not set above,
    # a service discovery action will be taken to find an instance of token service.
    # serviceId: ${client.tokenServiceId:com.networknt.oauth2-token-1.0.0}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.tokenProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.tokenProxyPort:3128}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # token endpoint for client credentials grant
    uri: /oauth2/token
    # client_id for client credentials grant flow.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client_secret for client credentials grant flow.
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
    # optional scope, default scope in the client registration will be used if not defined.
    # If there are scopes specified here, they will be verified against the registered scopes.
    scope:
    - market.r
    - market.w

```


With the serviceId identified, we can get jwk based on the config per serviceId. The following is the property in the values.yml file.


```
# The serviceId to the service specific OAuth 2.0 configuration. Used only when multipleOAuthServer is
# set as true. For detailed config options, please see the values.yml in the client module test.
serviceIdAuthServers: ${client.tokenKeyServiceIdAuthServers:}

```

The example in values.yml can be something like this. 


```
client.tokenKeyServiceIdAuthServers:
  com.networknt.petstore-3.0.1:
    # token server url. The default port number for token service is 6882. If this is set,
    # it will take high priority than serviceId for the direct connection
    server_url: https://localhost:6884
    # token service unique id for OAuth 2.0 provider. If server_url is not set above,
    # a service discovery action will be taken to find an instance of token service.
    # serviceId: ${client.tokenServiceId:com.networknt.oauth2-token-1.0.0}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.tokenProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.tokenProxyPort:3128}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # token endpoint for client credentials grant
    uri: /oauth2/key
    # client_id for client credentials grant flow.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client_secret for client credentials grant flow.
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
    # optional scope, default scope in the client registration will be used if not defined.
    # If there are scopes specified here, they will be verified against the registered scopes.
    # scope:
    # - petstore.r
    # - petstore.w
  com.networknt.market-1.0.0:
    # token server url. The default port number for token service is 6882. If this is set,
    # it will take high priority than serviceId for the direct connection
    server_url: https://localhost:6885
    # token service unique id for OAuth 2.0 provider. If server_url is not set above,
    # a service discovery action will be taken to find an instance of token service.
    # serviceId: ${client.tokenServiceId:com.networknt.oauth2-token-1.0.0}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.tokenProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.tokenProxyPort:3128}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # token endpoint for client credentials grant
    uri: /oauth2/key
    # client_id for client credentials grant flow.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # client_secret for client credentials grant flow.
    client_secret: f6h1FTI8Q3-7UScPZDzfXA
    # optional scope, default scope in the client registration will be used if not defined.
    # If there are scopes specified here, they will be verified against the registered scopes.
    # scope:
    # - petstore.r
    # - petstore.w

```







