---
title: "Self Ca Signed Cert"
date: 2018-03-02T07:03:44-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

### Self-signed vs CA-signed certificate

In light platform, there are two places that use certificates to ensure security for
microservices: 

* TLS connection

* JWT verification

A lot of questions have been received on when to use self-signed and when to use
commercial CA-signed certficate and here is the guideline from pure technical
perspective. 

* If the service is exposed to Internet, you have to use CA-signed certificate

This will make your service trustful and simplify the client side development
as they don't need to include the service public certificate in order to verify
the server certificate. 

If you cannot afford commercial certificate, there are some free offering and
one of the best would be [Let's Encrypt][]. 

* If the service is internal, it is safe to use self-signed certificate

Technically, there is no trust issue internally and the certificate is to ensure
that the connection is encrypted for confidentiality. A lot of organizations
uses self-signed certificates and big organizations might have their own CA.

For some of organizations like government and financial, they are using CA signed
certificate for every service regardless internal or external even on testing
environment. This is to follow certain security policy defined in the organization.

* Should I share a certificate between multiple services

If these services belong to the same application and have the same owener. Yes one
certificate can be shared and it make the configuration management much easier. It
is no recommended to share the same certifcate for services owned by two or more
groups even within the same organization. Different organization might have different
policy though. 

* When to rotate JWT certificate

Depending on how many JWT tokens has been signed, the JWT signature verification
certificate should be roated regularly. Usually once a year will be sufficent
enough if the OAuth 2.0 provider is deployed in house. light platform supports
JWT certificate roll out automatically in both light-oauth2 and light-4j JWT
verifier. For details, please refer to [key distribution][]

[Let's Encrypt]: https://letsencrypt.org/
[key distribution]: /architecture/key-distribution/