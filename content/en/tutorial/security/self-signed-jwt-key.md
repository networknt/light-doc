---
title: "Self Signed Jwt Key"
date: 2020-05-01T15:22:35-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


The light-oauth2 server needs a private key to sign the JWT token and all servers need a public key certificate to verify the token. The following includes the steps to create both primary and secondary key pairs. 

### keystore

```
keytool -genkeypair -alias primary -keyalg RSA -keysize 2048 -keystore primary.jks -storepass password -keypass password -validity 9999
keytool -genkeypair -alias secondary -keyalg RSA -keysize 2048 -keystore secondary.jks -storepass password -keypass password -validity 9999
```

Please note that light-oauth2 token service has a config file called jwt.yml and it looks like the following. The keyName in the config file must match the alias in the above command. 

```
key:
  kid: '100'                               # kid that used to sign the JWT tokens. It will be shown up in the token header.
  filename: "primary.jks"                  # private key that is used to sign JWT tokens.
  keyName: primary                         # key name that is used to identify the right key in keystore.
  password: password                       # private key store password and private key password is the same
issuer: urn:com:networknt:oauth2:v1        # default issuer of the JWT token
audience: urn:com.networknt                # default audience of the JWT token
expiredInMinutes: 10                       # expired in 10 minutes by default for issued JWT tokens
version: '1.0'                             # JWT token version
```



### certificate

```
keytool -export -alias primary -keystore primary.jks -rfc -file primary.crt
keytool -export -alias secondary -keystore secondary.jks -rfc -file secondary.crt

```

### deployment

Copy the primary.crt and secondary.crt to every server config folder and copy the primary.jks and secondary.jks to light-oauth2/token service config folder. 

