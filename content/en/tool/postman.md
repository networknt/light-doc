---
title: "Postman"
date: 2018-12-02T21:56:23-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

To test your HTTP service, the best tool is the Postman which has a GUI interface. The old version is running as part of the Chrome browser but the latest version is a standalone desktop application supports Linux, Mac and Windows. 

### Installation

To install it on Ubuntu Linux 18.04 and up, you can use snap. 

```
snap install postman
```

### Turn off Certificate Verification

As light-4j service has TLS enabled by default with a built-in self-signed certificate, you need either upload the certificate to the postman or simply turn off the certificate verification from settings/General/SSL certificate verification. 

### Upload the certificate

First, you need to export the certificate from client.truststore with Java keytool command and then you can upload from Settings/Certificates of the postman. 

