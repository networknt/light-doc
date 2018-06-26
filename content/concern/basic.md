---
title: "Basic Auth"
date: 2018-06-25T22:19:20-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---


As we know that the default security for the light platform is OAuth 2.0 and we have a provider light-oauth2 implemented as microservices. However, for some special use cases we do need to support other authentication mechanisms. One of them is basic authentication that is constantly asked by our customers who want to deploy the service to IoT devices. 

Unlike JWT verification, the basic authentication can be implemented as a generic middleware handler in light-4j instead of having different implementations for different frameworks as it has no dependencies to the framework. 


### Introduction

This is an optional middleware handler that is only used when OAuth 2.0 cannot be used. This handler should replace the default JwtVerifyHandler in the service.yml configuration. It basically take the Authorization header and decode it into username and password in order to compare with the usernam/password list. If the username or password is not matched, an error message will be returned. Otherwise, it will pass the control to the next handler in the chain. 

### Configuration

Here is an example of configuration basic.yml which has a flag to enable or disable it. Also, it contains a list of username/password pairs. Please note that the password can be encrypted with the [encryptor utility][]. 

```
# Basic Authentication Security Configuration for light-4j
---
# Enable Basic Authentication Handler, default is false.
enabled: false
# usernames and passwords, the password can be encrypted like secret.yml
# As we are supporting multiple users, so leave the passwords in this file.
users:
  - username: user1
    password: user1pass
  - username: user2
    password: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=
```

Although this middleware handler can support multiple users, there is no role concept for these users and all users should have the same access level. If you want to give different users different privileges, you can customized this handler. 

In order for this handler to be wire in the middleware handler chain, this handler need to be configured in service.yml middleware section. Here is an example. 

```
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Swagger Parsing swagger specification based on request uri and method.
  - com.networknt.swagger.SwaggerHandler
  # Basic Authentication middleware handler
  - com.networknt.basic.BasicAuthHandler
  # Body Parse body based on content type in the header.
  - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on swagger specification (depending on Swagger and Body)
  - com.networknt.validator.ValidatorHandler

```

If encryptor and decryptor are used, you need to provide Decryptor implementation in the service.yml as well.

```
# HandlerProvider implementation
- com.networknt.utility.Decryptor:
  - com.networknt.decrypt.AESDecryptor

```


### Error Messages

There are three possible errors for this handler. 

* MISSING_AUTH_TOKEN - The basic authentication header is missing.
* INVALID_BASIC_HEADER - The format of the basic authentication header is not valid.
* INVALID_USERNAME_OR_PASSWORD - Invalid username or password. 


[encryptor utility]: /tutorial/security/encrypt-decrypt/
