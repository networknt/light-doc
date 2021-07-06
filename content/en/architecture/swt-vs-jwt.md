---
title: "SWT vs JWT"
date: 2018-05-03T21:06:55-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In OAuth 2.0 [RFC6749][], the contents of tokens are opaque to clients and it is usually called simple web token(SWT). Most implementations choose UUID as SWT. This means that the client does not need to know anything about the content or structure of the token itself, if there is any. However, there is still a large amount of metadata that may be attached to a token, such as its current validity, approved scopes, and information about the context in which the token was issued. These pieces of information are often vital to protected resources making authorization decisions based on the tokens being presented. 

While the industry starts to adopt OAuth 2.0, there were two issues emerged. 

**No Token Introspection Standard**

Since OAuth 2.0 does not define a protocol for the resource server to learn meta-information about a token that it has received from an authorization server, different vendors implement the token introspections differently and this caused confusion for application developers. Also as resource server must speak different languages to different OAuth 2.0 providers, vendor lock-in happened. 

To standardize the token introspection, [RFC7662][] is published to define a method for a protected resource to query an OAuth 2.0 authorization server to determine the active state of an OAuth 2.0 token and to determine meta-information about this token. OAuth 2.0 deployments can use this method to convey information about the authorization context of the token from the authorization server to the protected resource. 

**SWT is not scaling**

For each request from a client to a resource server, the client needs to retrieve a token from OAuth 2.0 provider and send the request to resource server with the token. The resource server needs to send the token to OAuth 2.0 provider to introspect the token before granting the access. There are two extra network hops for each request, and OAuth 2.0 provider becomes a bottleneck and single point of failure. 

As original RFC6749 didn't specify the format of the token, several different approaches have been developed by vendors to carry authorization info in the token to allow resource servers to authorize the access without token intropsection. Most solutions used cryptographic algorithms to ensure that the token issued by the OAuth 2.0 provider is not compromised and from the authenticate OAuth 2.0 provider. 

To standardize this approach, the OAuth 2.0 specification working group published structured token formats JWT [RFC7519][]. The token contains a header, a payload and a signature and resource server can retrieve enough info to authorize the access locally without contacting the OAuth 2.0 provider. 


### Miscoservices architecture

As Light is aiming for microservices, the number of requests for service to service communication grows exponentially. There is no way we can use SWT in the OAuth 2.0 provider. So when we decided to implement microservices based [light-oauth2][], we chose the JWT for the token format. 

### JWT Pros and Cons

### SWT Pros and Cons


[RFC6749]: https://tools.ietf.org/html/rfc6749
[RFC7519]: https://tools.ietf.org/html/rfc7519
[RFC7662]: https://tools.ietf.org/html/rfc7662
[light-oauth2]: /service/oauth/

