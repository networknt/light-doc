---
title: "Reference"
date: 2017-12-07T11:42:35-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 40
weight: 40
aliases: []
toc: false
draft: false
reviewed: true
---

OAuth 2.0 is the industry-standard protocol for authorization. OAuth 2.0 supersedes the work done on the original OAuth protocol created in 2006. OAuth 2.0 focuses on client developer simplicity while providing specific authorization flows for web applications, desktop applications, mobile phones, and living room devices. This specification and its extensions are being developed within the IETF OAuth Working Group.



## Specifications


* [OpenID Connect Core 1.0 incorporating errata set 1](http://openid.net/specs/openid-connect-core-1_0.html#OAuth.Responses) - OpenID Connect core
* [OAuth 2.0 Multiple Response Type Encoding Practices](http://openid.net/specs/oauth-v2-multiple-response-types-1_0.html) - OpenID Connect multiple response types 
* [OAuth 2.0 Form Post Response Mode](http://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html) - Form post response mode
* [OAuth 2.0 Threat Model and Security Considerations](https://tools.ietf.org/html/rfc6819) - Information on all sorts of threats in OAuth 2.0 specification.
* [JSON Web Key (JWK)](https://tools.ietf.org/html/rfc7517) - Enable federated OAuth 2.0 provider clusters.
* [OAuth 2.0 Token Introspection](https://tools.ietf.org/html/rfc7662) - Used as reference to implement de-reference opaque token to JWT.
* [OAuth 2.0 Token Revocation](https://tools.ietf.org/html/rfc7009) - Revoke access token and refresh token when they are compromised. 


## Articles and Blogs

* [OAuth 2 and Fragment encoding](http://www.thread-safe.com/2014/05/oauth-2-and-fragment-encoding.html) - Some browsers changed behavior for fragment encoding and the impact on OAuth 2.0
* [Open Redirect Vulnerability](https://weblog.bulknews.net/covert-redirect-vulnerability-with-oauth-2-2c1f3083b1b4#.t1ldg8xf8) - A vulnerability that requires OAuth2 provider to validate redirect_uri.


