---
title: "Swt Verifier"
date: 2023-03-15T10:51:47-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

A simple web token is used instead of a JSON web token for some external requests, and the light-4j server needs to send the token to the OAuth 2.0 provider for token introspection. In this case, the SwtVerifier will be used instead of [JwtVerifier][]. 


The SWT introspection configuration is the same as the JWK configuration, and they share the same token key section in the client.yml file. The only difference is that the client_id and client_secret might be required for the introspection but not JWK. 

The SWT security middleware handler can be enabled as a standalone handler in the handler.yml, or UnifiedSecurityHandler can invoke it. 

### Standalone

If only one OAuth 2.0 provider is configured on the server, and only SWT is supported, you can add the following section in the values.yml to enable SWT introspection. 

```
# client.yml

client.tokenKeyServerUrl: http://localhost:7080
client.tokenKeyUri: /oauth/introspection
client.tokenKeyClientId: f7d42348-c647-4efb-a52d-4c5787421e72
client.tokenKeyClientSecret: f6h1FTI8Q3-7UScPZDzfXA
client.tokenKeyEnableHttp2: true

```

By default, the SWT is not enabled in the openapi-security.yml configuration. You need to enable it and possibly disable the JWT verifier. If the client_id and client_secret are not in the above client.yml configuration. For example, an external consumer will pass the client_id and client_secret from the request header. You can define the header names for the client_id and client_secret. If you are using the swt-client and the swt-secret as the header names, you can skip these header definitions as they are the default values. 

```
# openapi-security.yml

openapi-security.enableVerifyJwt: false
openapi-security.enableVerifySwt: true

openapi-security.swtClientIdHeader: swt-client
openapi-security.swtClientSecretHeader: swt-secret

```

You also need to add the SWT handler to the handlers section and use it in the default chain in the handler.yml file. 


### UnifiedSecurity



[JwtVerifier]: /concern/jwt-verifier/



