---
title: "Security Tutorial"
date: 2017-12-11T10:46:07-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

The light platform is following security first design as most of our customers are big financial companies and they need to ensure confidentiality, availability, and integrity first then scale the system to lower the production cost and development cost. 

When the platform was designed, we focused a lot of efforts on the security details and made a lot of default configuration so that our customers won't need to do extra things to bring up a system securely. Once they feel comfortable, they can start to configure their services based on their business requirement or non-functional requirement. 

In this section, we have several tutorials related to security configurations and implementations. 

* It is essential to understand the [basic terminologies and formats of certs and keys][] before proceeding other tutorials.

* How to set up your service to [bootstrap from light-oauth2 key service][] for JWT signature verification public key certificate distribution.

* How to [encrypt and decrypt sensitive info][] from secret.yml or other config files. 

* How to use [keystore and truststore][] for one-way and two-way TLS.

* How to create keystore for the [light-config-server TLS][] and bootstrap.truststore for light-4j service.

* A real example on [adding server public key certificate to client.truststore][] to access public service on the Internet with HTTPS.

* How to [install a self-signed or commercial CA-signed server certificate][] to light-4j service

* How to get free [Let's Encrypt certificate][] for public-facing light-4j secd rvice

* How to [convert CA-signed certificate to server.keystore][] that can be used by light-4j service

* Running [light-4j service on port 443][] using iptables

[bootstrap from light-oauth2 key service]: /tutorial/security/bootstrap-from-key-service/
[encrypt and decrypt sensitive info]: /tutorial/security/encrypt-decrypt/
[keystore and truststore]: /tutorial/security/keystore-truststore/
[adding server public key certificate to client.truststore]: /tutorial/security/publickey-truststore/
[Install a self-signed or commercial CA-signed server certificate]: /tutorial/security/install-certificate/
[Let's Encrypt certificate]: /tutorial/security/lets-encrypt/
[convert CA-signed certificate to server.keystore]: /tutorial/security/ca-certificate/
[basic terminologies and formats of certs and keys]: /tutorial/security/term-format/
[light-4j service on port 443]: /tutorial/security/port443/
[light-config-server TLS]: /tutorial/security/config-server-cert/
