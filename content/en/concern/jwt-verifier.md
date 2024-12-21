---
title: "Jwt Verifier"
date: 2023-03-15T10:51:31-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

This is a class in the security module in the light-4j repository. It is responsible for verifying a JWT token passed from the authorization header in a request. If your consumer is using a simple web token, please refer to the [SwtVerifier][] handler.

The following validations are performed in the handler: 

### Expiration

### Signature

### Audience

For some cloud OAuth 2.0 providers, there is only one tenant for an organization with multiple APIs and consumers. To support multiple environments, we need to validate the JWT token audience to ensure there is no cross-fire in the token validation. 

The audience validation will be enabled if there is a client.tokenKeyAudience defined in the values.yml

```
# client.yml
client.tokenKeyAudience: abc
```

The above property is part of the Jwk configuration. And it can be configured with multiple OAuth 2.0 providers. 

Because the configuration is part of the Jwk, it can only be enabled if jwk is set as the resolver and bootstrapFromKeyService is set to true. 


```
# security.yml
security.keyResolver: JsonWebKeySet
security.bootstrapFromKeyService: true
```

During the server startup, the audience or a map of the audience from the serviceId will be loaded from the client.yml configuration file. 

If UnifiedSecurity is used, there is a list of serviceIds will be passed in to find the configured audience(s) to make the comparison. Otherwise, a serviceId will be passed in to look up the audience for comparison. 

In the Jwt token, we always assume there is a list of audiences in the claims. If the list has only one element, we get it with zero index and compare it with the configured audience. 

If there is a list of audiences in the claims, we need to compare with each audience to confirm the token's validity.

[SwtVerifier]: /concern/swt-verifier/
