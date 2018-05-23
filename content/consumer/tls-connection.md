---
title: "TLS Connection"
date: 2018-03-01T21:36:44-05:00
description: ""
categories: []
keywords: []
weight: 20
aliases: []
toc: false
draft: false
---

Light platform supports One-Way SSL by default in the light-codegen and Two-Way SSL by updating service.yml to enable. Unless you are using some old tools that doesn't support HTTPS, it is recommended to use at least One-Way SSL even in the develop phase so you don't have any suprise when release to test environment. 

### TLS certificates

There are four keystore files can be generated from light-codegen depending on the config.json in model-config repository

Here is an example. 

```
{
  "name": "apia",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "apia",
  "rootPackage": "com.networknt.apia",
  "handlerPackage":"com.networknt.apia.handler",
  "modelPackage":"com.networknt.apia.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 7001,
  "enableHttp": true,
  "httpsPort": 7441,
  "enableHttps": true,
  "enableRegistry": false,
  "supportDb": false,
  "supportH2ForTest": false,
  "supportClient": true
}

```

By default, the generated code will have server.keystore and server.truststore in the config folder. But if supportClient is true in config.json, then client.keystore and client.truststore will be generated as well. 

For information about keystore files, please refer to [keystore truststore][]. 

The generated keystores and truststores contains self-signed certificates expire in the year 2023 and these should be used for development only. Once move to an official test environment, they need to be replaced with other self-signed certificates or CA-signed certificates.  

Please refer to [self-signed vs. CA-signed certificate][] for details on when to use self-signed or CA-signed certificate. 

### Enable One-Way or Two-Way TLS

Please refer to the [server config][] for more details.

[keystore truststore]: /tutorial/security/keystore-truststore/
[self-signed vs CA-signed certificate]: /faq/self-ca-signed-cert/
[server config]: /concern/server/
