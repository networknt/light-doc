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
---

When working with microservices, the chances are that sensitive data need to be passed through several services before reaching the target services. For example, credit card number is collected in the shopping cart service and needs to pass through order service to reach the payment service. To make your application [PCI compliant][], the credit card number should be encrypted or tokenized at the shopping cart service and only payment service can decrypt or detokenize it. 

Another business scenario is that when information passed to the Internet, sensitive info needs to be protected for confidentiality. Again, there are two approaches for it: encryption and tokenization. 

The light platform has [encryptor and decryptor][] for data encryption as plugins that support custom implementations. It covers some use cases for data confidentiality, and tokenization covers the rest. 

In the light platform, tokenization service is provided as part of the enterprise package and hosting service. It is built on top of light-4j and light-rest-4j frameworks with very high throughput and very low latency as most operations are done in the cache. 

If you are interested in deploy this service in your data center or sign up our hosting service, please contact sales@networknt.com for details. 


* [Getting Started][] with our test service and tutorial
* [API Document] to learn the details on how it works
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* 

[PCI compliant]: http://www.onlinetech.com/resources/references/what-is-pci-compliance
[encryptor and decryptor]: /concern/decryptor/
[Getting Started]: /service/tokenization/getting-started/
[Artifact]: /service/tokenization/artifact/
[Configuration]: /service/tokenization/configuration/
[API Document]: /service/tokenization/document/
