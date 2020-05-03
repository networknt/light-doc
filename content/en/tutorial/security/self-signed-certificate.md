---
title: "Self Signed Certificate"
date: 2020-05-01T15:22:11-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In other tutorials, we have introduced Java keytool and other tools for certificate generation and manipulation. A lot of users still cannot grasp the complicated procedure and ask if we can provide a tutorial to list all the steps without too many options. 

The following is a list of steps for us to create a production API. If you have a list of APIs group together, they can share the same server.keystore and client.truststore pair. 

The following steps assume one-way TLS is used.

### Create server.keystore

```
keytool -genkeypair -alias prod -keyalg RSA -keysize 2048 -keystore server.keystore -storepass password -keypass password -validity 9999
```

Please change the storepass and keypass as well as alias accordingly.

### List content of the server.keystore

```
keytool -list -v -keystore server.keystore
```

### Export the certificate

```
keytool -export -alias prod -keystore server.keystore -rfc -file prod.cert
```

### Create a client.truststore

```
keytool -importcert -alias prod -file prod.cert -keystore client.truststore
```

### Deployment

Copy the server.keystore to config folder for all APIs.

Copy the client.truststore to config folder for all APIs that invoke other APIs. 

Update client.yml and server.yml accordingly for passwords. 



