---
title: "API Key"
date: 2022-11-03T23:07:58-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For some legacy applications to migrate from a monolithic gateway to the light-gateway without changing any code on the consumer application, we need to support the API Key authentication/authorization on the light-gateway(LG) or client-proxy(LCP). The consumer application sends the API key in a header to authenticate/authorize itself on the light-gateway. Then the light-gateway will retrieve a JWT token to access the downstream API. 

Only specific paths will have API Key set up, and the header name for each application might be different. To support all use cases, we add a list of maps to the configuration apikey.yml to pathPrefixAuths property. 


Each config item will have pathPrefix, headerName and apiKey. The handler will try to match the path prefix first and then get the input API Key from the header. After comparing with the configured API Key, the handler will return either ERR10057 API_KEY_MISMATCH or pass the control to the next handler in the chain. 

### Configuration

Here is the built-in configuration template. 

```
# ApiKey Authentication Security Configuration for light-4j
# Enable ApiKey Authentication Handler, default is false.
enabled: ${apikey.enabled:false}
# path prefix to the api key mapping. It is a list of map between the path prefix and the api key
# for apikey authentication. In the handler, it loops through the list and find the matching path
# prefix. Once found, it will check if the apikey is equal to allow the access or return an error.
# The map object has three properties: pathPrefix, headerName and apiKey. Take a look at the test
# resources/config folder for configuration examples.
pathPrefixAuths: ${apikey.pathPrefixAuths:}
```

By default, this handle is disabled. So you can safely include it in the default request/response chain. It can only be turned on when you add the following property to your values.yml file. 

```
# apikey.yml
apikey.enabled: true
```

Even if you have this handler enabled, this handler might not be invoked to check the API Key for your request. Your request path must be in the configuration pathPrefixAuths list to ensure that this handler checks API Key. The server will skip this handler if there is no definition for the pathPrefixAuths in the values.yml file. 


Here is one way to define the property in YAML format in values.yml file.

```
apikey.pathPrefixAuths:
  - pathPrefix: /test1
    headerName: x-gateway-apikey
    apiKey: abcdefg
  - pathPrefix: /test2
    headerName: x-apikey
    apiKey: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=
```

Here is the way to define the pathPrefix, headerName and apiKey mapping in JSON format in values.yml file. It is suitable for the config server as the entire object can be constructed as a string. 

```
apikey.pathPrefixAuths: [{"pathPrefix":"/test1","headerName":"x-gateway-apikey","apiKey":"abcdefg"},{"pathPrefix":"/test2","headerName":"x-apikey","apiKey":"CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w="}]

```

Most services will have multiple consumers, and each consumer might have its own API key for authentication. In this case, we can have multiple entries of the same path prefix in the pathPrefixAuths. 

```
apikey.pathPrefixAuths:
  - pathPrefix: /test1
    headerName: x-gateway-apikey
    apiKey: abcdefg
  # The same prefix has another apikey header and value.
  - pathPrefix: /test1
    headerName: authorization
    apiKey: xyz
  - pathPrefix: /test2
    headerName: x-apikey
    apiKey: CRYPT:3ddd6c8b9bf2afc24d1c94af1dffd518:1bf0cafb19c53e61ddeae626f8906d43
```

With the above configuration in the values.yml on the gateway, the service with path prefix /test1 has two consumers and two different API keys from different headers. 

### Error Response

This handler only returns an error response ERR10075 if the pathPrefix and the header are matching and the apiKey is not equal between the configured value and the header value. 


Here is the error message defined in the status.yml config file.

```
ERR10075:
  statusCode: 401
  code: ERR10075
  message: API_KEY_MISMATCH
  description: APIKEY from header %s is not matched for request path prefix %s.
```

To prevent leaking sensitive information, the apiKey from the config file is not revealed in the error message. 

### Usage

Once you configure this handler in the values.yml file, you need to add it to the request/response chain to allow it to work. 

In the handler.yml or the values.yml if you have externalized the handlers definition and default chain. 


```
handler.handlers:
  .
  .
  .
  - com.networknt.apikey.ApiKeyHandler@apikey

handler.chains.default:
  .
  .
  .
  - apikey
  - security
  .
  .
  .

```

This is the [PR](https://github.com/networknt/light-gateway/issues/92) on how we add the apikey handler to the chain in the light-gateway values.yml file. 

