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

The light platform supports One-Way SSL by default in the [light-codegen][] and Two-Way SSL by updating server.yml to enable. Unless you are using some old tools that don't support HTTPS, it is recommended to use at least One-Way SSL even in the development phase, so you don't have any surprise when releasing to an official test environment. 

### TLS certificates

There are four keystore files can be generated from light-codegen depending on the config.json in the model-config repository.

Here is an example of config.json for light-codegen. 

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

The generated keystores and truststores contains self-signed certificates expire in the year 2023, and these should be used for development only. Once move to an official test environment, they need to be replaced with other self-signed certificates or CA-signed certificates.  

Please refer to [self-signed vs. CA-signed certificate][] for details on when to use self-signed or CA-signed certificate. 

### Enable One-Way or Two-Way TLS

Please refer to the [server config][] for more details.

### Download certificate from the server

While connecting to a server with HTTPS, you should ask for the client certificate from the server admin. If you cannot get the certificate from the server admin, you can download it from the server with openssl.

Please refer to the [public https][] tutorial for more details. 

### Debug TLS connection

When make TLS connection to the server, you need to add certificates into client.truststore most of the cases. For most developers, it might be a challenge to get it done right in the first place. If you connection is not established to the server, chances are that you have the client.truststore missing the client certifiate. To figure out if the connection issue is due to the certificate, you can enable the tls debug in your IDE. 

Here is an article that contains all the details on [Debugging SSL/TLS Connections][].

Baiscally, you need to put the following JVM option when starting your server in IDE. 

```
-Djavax.net.debug=all
```

[keystore truststore]: /tutorial/security/keystore-truststore/
[self-signed vs CA-signed certificate]: /faq/self-ca-signed-cert/
[server config]: /concern/server/
[light-codegen]: /tool/light-codegen/
[public https]: /tutorial/client/public-https/
[Debugging SSL/TLS Connections]: https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html

