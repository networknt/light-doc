---
title: "Self or CA Signed Cert"
date: 2018-03-02T07:03:44-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

### Self-signed vs CA-signed certificate

In Light, two places use certificates to ensure security for microservices: 

* TLS connection

* JWT verification

A lot of questions have been asked on when to use a self-signed certificate and when to use a commercial CA-signed certificate. And here is the guideline from a purely technical perspective. 

* If the service is exposed to the Internet, you have to use a CA-signed certificate

It will make your service trustful and simplify the client side development as they don't need to include the service public certificate to verify the server certificate. 

If you cannot afford commercial certificate, there are some free offerings and one of the best would be [Let's Encrypt][]. 

* If the service is internal, it is safe to use a self-signed certificate.

Technically, there is no trust issue internally, and the certificate is to ensure that the connection is encrypted for confidentiality. A lot of organizations use self-signed certificates, and big organizations might have their own CA. We are working on an integration with a leading open source CA product to help our customers to issue and manage certificate. 

For some organizations like the government and financial organizations, they are using the commercial CA-signed certificates for every service regardless of internal or external even on the testing environment. It is to follow the specific security policy defined in the organization.

* Should I share a certificate between multiple services?

If these services belong to the same application and have the same owner, one certificate can be shared, and it makes the configuration management much more manageable. It is not recommended to share the same certificate for services owned by two or more groups even within the same organization. A different organization might have a different policy though. 

* When to rotate JWT certificate

Depending on how many JWT tokens have been signed, the JWT signature verification certificate should be rotated regularly. Usually, once a year will be sufficient enough if the OAuth 2.0 provider is deployed in-house. Light supports JWT certificate roll out automatically in both light-oauth2 and light-4j JWT verifier. For details, please refer to [key distribution][]

[Let's Encrypt]: https://letsencrypt.org/
[key distribution]: /architecture/key-distribution/