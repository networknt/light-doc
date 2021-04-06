---
title: "Tokenization"
date: 2018-03-22T21:10:56-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 70
weight: 70
aliases: []
toc: false
draft: false
reviewed: true
---

When working with microservices, chances are that sensitive data must be passed through several services before reaching the target service. For example, a credit card number is collected in the shopping cart service and needs to pass through order service to reach the payment service. To make the application [PCI compliant][], the credit card number should be encrypted or tokenized at the shopping cart service so that only the payment service can decrypt or detokenize it.

Another business scenario is that when information is passed to the Internet, sensitive information needs to be protected for confidentiality. For example, the account number in the URL needs to be hidden so that nobody can dig it from log files. Again, there are two approaches for it: encryption and tokenization. 

Light has an [encryptor and decryptor][] for data encryption as plugins that support custom implementations. It covers some use cases for data confidentiality, and tokenization covers the rest.

Tokenization is a reversible security method that replaces sensitive data with fake data that looks and feels just like the real thing while making it worthless to potential thieves. Tokenization can provide equal or better security than encryption while retaining the vital usability of data for analytics and other business processes.

Flexible, format-preserving token types, including numeric, alphanumeric, date, time, address, and other structured tokens can be created with “bleed through” with parts of the original data exposed for business purposes, preserving privacy when applications require only part of the sensitive data for processing.

In Light, tokenization service is provided as part of the enterprise package and hosting service. It is built on top of light-4j and light-rest-4j frameworks with very high throughput and very low latency as most operations are done in the cache. It also supports multi-tenancy based on the client_id in JWT token. A group of clients can share the same token vault, or each client can have its vault.

If you are interested in deploying this service in your data center or signing up our hosting service, please contact sales@lightapi.net for details.


* [Getting Started][] with our test service and tutorial
* [API Document] to learn the details on how it works
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* [Performance][] for production like environment with both TLS and OAuth 2.0 enabled.
* [Reference][] gives users more information about the standards and other implementations. 
* [Roadmap][] shows which areas that need to be enhanced in the future.  

[PCI compliant]: http://www.onlinetech.com/resources/references/what-is-pci-compliance
[encryptor and decryptor]: /concern/decryptor/
[Getting Started]: /service/tokenization/getting-started/
[Artifact]: /service/tokenization/artifact/
[Configuration]: /service/tokenization/configuration/
[API Document]: /service/tokenization/document/
[Performance]: /service/tokenization/performance/
[Reference]: /service/tokenization/reference/
[Roadmap]: /service/tokenization/roadmap/
