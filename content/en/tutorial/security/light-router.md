---
title: "Light Router"
date: 2022-01-23T20:38:40-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When deploying the light-portal to the cloud, the light-config-server won't be accessed directly by the services deployed to the public cloud. Services will access the config server through the light-router instance which is the entry point for all the backend platform services. 

All services will access the config server with bootstrap.truststore to load the configuration from the config-server during the startup. It requires that the light-config-server private key will be imported to the light-router server.keystore. 

Also, the light-router will access the light-config-server for the portal UI function, so we need to import the light-config-server cert to the light-router client.truststore. 


### import public key cert

It is easy to export the cert from the light-4j/server/src/main/resources/config/bootstrap.trustore with alias config-server and import it into the client.truststore of the light-router. Follow the instruction from the [keystore-truststore][].

### import private key

If the keystore of the light-config-server is jks format, you need to export it to standardized format of PKCS12 format. 

```
keytool -importkeystore -srckeystore server.keystore -destkeystore server.keystore -deststoretype pkcs12
```

The default server.keystore format in the light-config-server is P12 format, so we just follow the command below to 
copy the server.keystore to the light-router src/main/resources/config folder where the server.keystore located.

```
cd ~/networknt/light-router/src/main/resources/config
cp ~/networknt/light-config-server/src/main/resources/config/server.keystore config-server.keystore
keytool -importkeystore -srckeystore config-server.keystore -destkeystore server.keystore -deststoretype JKS
```

After the import, there should be two entries in the server.keystore file. Remove the config-server.keystore after the import. 


[keystore-truststore]: /tutorial/security/keystore-truststore/
