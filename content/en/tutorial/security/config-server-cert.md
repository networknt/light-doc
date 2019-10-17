---
title: "Config Server Certificate"
date: 2019-10-17T13:48:38-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

For light-4j services, there are four options to get the configuration values for each handler or cross-cutting concerns. 

* externalized configuration files with all the final values.
* externalized configuration files with values.yml to inject into the config file templates. 
* externalized configuration files with system properties to inject into the config file templates.
* config files/values served by light-config-server instance.

When option four is chosen, we need to ensure that the service can connect to the light-config-server instance(s) with a secure channel. It requires that the light-config-server is HTTPS and HTTP/2 enabled, and a bootstrap certificate or bootstrap.truststore is installed on the service side. 

This tutorial shows you how to create a server.keystore for the light-config-server and bootstrap.truststore for services to connect to the light-config-server. 

### server.keystore

We need to first create a server.keystore for the one-way TLS of the light-config-server. The file will be located in the main/resources/config folder by default. In a production deployment for the light-config-server, you need to create a new self-signed certificate or commercial certificate and externalize it to replace the default certificate. You can follow the exact steps in this tutorial to do so. 

We use the Java keytool to create the server.keystore with the following command line. 

```
cd ~/networknt/light-config-server/src/main/resources/config
keytool -genkeypair -keystore server.keystore -keyalg RSA -alias config-server -dname "CN=localhost OU=OU O=Org L=City ST=State C=GB" -storepass password -keypass password -validity 3950
```

You can use the list command to check the Keystore. 

```
keytool -list -v -keystore server.keystore
```

### config-server.cer

Let's export the public key certificate from the server.keystore and use it to create the bootstrap.truststore for services that connect to the light-config-server instance. 

```
keytool -exportcert -alias config-server -keystore server.keystore -rfc -file config-server.cer
```

You should find a file named config-server.cer in the current folder. 

### bootstrap.truststore

With the newly created config-server.cer, let's create bootstrap.truststore

```
keytool -importcert -keystore bootstrap.truststore -alias config-server -storepass password -file config-server.cer -noprompt
```

The bootstrap.truststore is not supposed to be saved into light-config-server. Let's copy it to the light-4j server module. And we don't need the certificate file config-server.cer anymore.

```
mv bootstrap.truststore ~/networknt/light-4j/server/src/main/resources/config
rm config-server.cer
```

By default, we just copied the bootstrap.truststore into the server module in light-4j. In production, you need to replace it with an externalized file created from your certificate of the light-config-server. 

