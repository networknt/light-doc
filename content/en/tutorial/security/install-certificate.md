---
title: "Install Certificate"
date: 2018-10-29T08:06:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For all light-4j instances, it is highly recommended using HTTPS/HTTP2 instead of HTTP unless your infrastructure doesn't support HTTPS. That is why HTTPS/2 are enabled right out of the light-codegen with a set of built-in [keystores and truststores][] for development. Before moving to an official testing environment, you need to replace the self-signed certificates with your self-signed or commercial certificates. Please be aware that client.yml, client.keystore, and client.truststore will only be generated when you have the following config attribute in the config.json for light-codegen input. You need to enable this flag if your service is going to call other services in the call stack. 

```
"supportClient": true,
```

If you are unsure what kind of certificate should be used in your use case, please refer to [Self-signed vs CA-signed certificate][] for more info. From the process perspective, there is no difference between a CA-signed or self-signed certificate. 

The following tutorial will only focus on the server side configuration. If you want to setup client-side certificate, please follow [adding server certificate to client.truststore][] for one-way TLS. 

You can use the existing server.keystore and replace the default certificate with the new one, but we don't recommend it as the process is a little bit more complicated and error-prone. It is easier to create your server.keystore with the certificate you have. Different vendors will have different formats with their issued certificate. Sometimes you will get two files, and sometimes you will get three files. There would be a lot of tutorials on the Internet on how to use Java keytool to create .keystore with varying formats of the certificate. You vendor might have a guideline for Java application as well. If you are using [Let's Encrypt][], then you can follow this [CA-signed certificate][] tutorial to create a server.keystore. 

If you want a quick reference of Java keytool, please refer to [keytool][]. 

Often, the public tutorial will use .jks as the file extension. It is OK for one-way TLS but might be confusing if two-way TLS is in use. It is a good idea to stay with the convention of light-4j and name your file as server.keystore. 


Once the file is created, copy it into the config folder and update the server.yml to specify the filename. 

```
# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore
```

During the creation of server.keystore, two passwords need to be captured. They need to be input into the secret.yml file. 

```
# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

```

If you config folder is externalized, then you need to restart your service. Otherwise, you need to rebuild the server and restart it to ensure that the new certificate is used. 

[keystores and truststores]: /tutorial/security/keystore-truststore/
[Self-signed vs CA-signed certificate]: /faq/self-ca-signed-cert/
[adding server certificate to client.truststore]: /tutorial/security/publickey-truststore/
[keytool]: /tool/keytool/
[Let's Encrypt]: /tutorial/security/lets-encrypt/
[CA-signed certificate]: /tutorial/security/ca-certificate/
