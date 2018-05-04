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

In OAuth 2.0 [RFC6749][], the contents of tokens are opaque to clients and it is usually called simple web token(SWT). This means that the client does not need to know anything about the content or structure of the token itself, if there is any. However, there is still a large amount of metadata that may be attached to a token, such as its current validity, approved scopes, and information about the context in which the token was issued.  These pieces of information are often vital to protected resources making authorization decisions based on the tokens being presented. Since OAuth 2.0 does not define a protocol for the resource server to learn meta-information about a token that it has received from an authorization server, several different approaches have been developed to bridge this gap.  These include using structured token formats such as JWT [RFC7519][] or proprietary inter-service communication mechanisms (such as shared databases and protected enterprise service buses) that convey token information.

To standardize the token introspection, [RFC7662][] is published to define a method for a protected resource to query an OAuth 2.0 authorization server to determine the active state of an OAuth 2.0 token and to determine meta-information about this token. OAuth 2.0 deployments can use this method to convey information about the authorization context of the token from the authorization server to the protected resource. 




[RFC6749]: https://tools.ietf.org/html/rfc6749
[RFC7519]: https://tools.ietf.org/html/rfc7519
[RFC7662]: https://tools.ietf.org/html/rfc7662

