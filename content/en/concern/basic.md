---
title: "Basic Auth"
date: 2018-06-25T22:19:20-04:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

We know that the default security for Light is OAuth 2.0, and we have a provider, light-oauth2, to support it. However, we must support other authentication mechanisms for some exceptional use cases. One of them is basic authentication, which is constantly asked about by our customers who want to deploy the service to IoT devices. Or use it in the light-gateway to integrate legacy services into the light-4j ecosystem. 

Unlike JWT verification, the basic authentication can be implemented as a generic middleware handler in light-4j instead of having different implementations for different frameworks, as it has no dependencies on any framework. 

### Introduction

This optional middleware handler is only used when OAuth 2.0 JWT or SWT cannot be used. This handler should replace the default JwtVerifyHandler in the configuration of the handler.yml request/response chain. It takes the Authorization header and decodes it into a username and password to compare with the username/password list from the configuration. An error message will be returned if the username or password is not matched. Otherwise, it will pass the control to the next handler in the chain. 


### Configuration

Here is the default configuration basic-auth.yml, which has a flag to enable or disable it. It also contains a list of username/password pairs. Please note that the password can be encrypted with the [encryptor utility][].

```
# Basic Authentication Security Configuration for light-4j
---
# Enable Basic Authentication Handler, default is false.
enabled: ${basic.enabled:false}
# Enable Ldap Authentication, default is true.
enableAD: ${basic.enableAD:true}
# Do we allow the anonymous to pass the authentication and limit it with some paths
# to access? Default is false, and it should only be true in client-proxy.
allowAnonymous: ${basic.allowAnonymous:false}
# Allow the Bearer OAuth 2.0 token authorization to pass to the next handler with paths
# authorization defined under username bearer. This feature is used in proxy-client
# that support multiple clients with different authorizations.
allowBearerToken: ${basic.allowBearerToken:false}
# usernames and passwords in a list, the password can be encrypted like user2 in test.
# As we are supporting multiple users, so leave the passwords in this file with users.
# For each user, you can specify a list of optional paths that this user is allowed to
# access. A special user anonymous can be used to set the paths for client without an
# authorization header. The paths are optional and used for proxy only to authorize.
users: ${basic.users:}

```

The following is an example of externalized basic-auth.yml file in your application. 

```
# Basic Authentication Security Configuration for light-4j
---
# Enable Basic Authentication Handler, default is false.
enabled: ${basic.enabled:true}
# Do we allow the anonymous to pass the authentication and limit it with some paths
# to access? Default is false, and it should only be true in client-proxy.
allowAnonymous: ${basic.allowAnonymous:false}
# Allow the Bearer OAuth 2.0 token authorization to pass to the next handler with paths
# authorization defined under username bearer. This feature is used in proxy-client
# that support multiple clients with different authorizations.
allowBearerToken: ${basic.allowBearerToken:false}
# usernames and passwords, the password can be encrypted like user2
# As we are supporting multiple users, so leave the passwords in this file.
# For each user, you can specify a list of paths that this user is allowed
# a special user anonymous can be used to set the paths for client without
# Basic auth and Bearer auth. The paths are optional and used for proxy only.
users:
  - username: user1
    password: user1pass
    paths:
      - /v1/address
  - username: user2
    password: CRYPT:0754fbc37347c136be7725cbf62b6942:71756e13c2400985d0402ed6f49613d0
    paths:
      - /v2/pet
      - /v2/address
      - /v2/party
  # a special user to indicate the Authorization header does not exist.
  - username: anonymous
    paths:
      - /v1/party
      - /info

```

Usually, you don't externalize the entire file. You can overwrite some properties from the module's built-in configuration file in the values.yml file. 

values.yml
```
basic.enabled: true
basic.users:
  - username: user1
    password: user1pass
    paths:
      - /v1/address
  - username: user2
    password: CRYPT:0754fbc37347c136be7725cbf62b6942:71756e13c2400985d0402ed6f49613d0
    paths:
      - /v2/pet
      - /v2/address
      - /v2/party
  # a special user to indicate the Authorization header does not exist.
  - username: anonymous
    paths:
      - /v1/party
      - /info

```

Although this middleware handler can support multiple users, there is no role concept for these users and all users should have the same access level. If you want to give different users different privileges, you can customized this handler. 

In order for this handler to be wired in the middleware handler chain, this handler needs to be configured in the handler.yml 

```
handler.handlers:
  .
  .
  .
  - com.networknt.basicauth.BasicAuthHandler@basic

handler.chains.default:
  .
  .
  .
  - basic
  .
  .
  .

```

### Error Messages

There are three possible errors for this handler. 

* MISSING_AUTH_TOKEN - The basic authentication header is missing.
* INVALID_BASIC_HEADER - The format of the basic authentication header is not valid.
* INVALID_USERNAME_OR_PASSWORD - Invalid username or password. 

### Unified Security

When light-gateway is used for multiple consumers and providers, chances are API Key, Basic, JWT, and SWT security will be used by different services. And one service might use several security handlers simultaneously to allow different consumers to authenticate and authorize themselves. In this case, the API Key handler can join the [UnifiedSecurityHandler][] for a unified security solution. 

 
[UnifiedSecurityHandler]: /service/gateway/unified-security/
[encryptor utility]: https://github.com/networknt/light-encryptor
