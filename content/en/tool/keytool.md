---
title: Java keytool
date: 2017-11-04T22:12:19-04:00
description: "Java keytool is used to generate key pairs for TLS and JWT"
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---


Java Keytool is a key and certificate management utility. It allows users to manage their public/private key pairs and certificates. It also allows users to cache certificates. Java Keytool stores the keys and certificates in what is called a keystore. By default, the Java keystore is implemented as a file. It protects private keys with a password. A Keytool keystore contains the private key and any certificates necessary to complete a chain of trust and establish the trustworthiness of the primary certificate.

Each certificate in a Java keystore is associated with a unique alias. When creating a Java keystore you will first create the .jks file that will initially only contain the private key. You will then generate a CSR and have a certificate generated from it. Then you will import the certificate to the keystore including any root certificates. Java Keytool also several other functions that allow you to view the details of a certificate or list the certificates contained in a keystore or export a certificate.

In light-4j platform, both one-way and two-way TLS are supported. There are four files that might be used: 

* server.keystore
* server.truststore
* client.keystore
* client.truststore

Please refer to the [keystore truststore][] for more info about how to use these files. 

### Keytool Usage

* Java Keytool Commands for Creating and Importing

These commands allow you to generate a new Java Keytool keystore file, create a CSR, and import certificates. Any root or intermediate certificates will need to be imported before importing the primary certificate for your domain.

Generate a Java keystore and key pair

```
keytool -genkey -alias mydomain -keyalg RSA -keystore server.keystore -keysize 2048
```

Generate a keystore and certificate

```
keytool -genkey -alias mycert -keyalg RSA -sigalg MD5withRSA -keystore server.keystore -storepass secret  -keypass secret -validity 9999
```

Generate a certificate signing request (CSR) for an existing Java keystore

```
keytool -certreq -alias mydomain -keystore keystore.jks -file mydomain.csr
```

Import a root or intermediate CA certificate to an existing Java keystore

```
keytool -import -trustcacerts -alias root -file Thawte.crt -keystore client.truststore
```

Import a signed primary certificate to an existing Java keystore

```
keytool -import -trustcacerts -alias mydomain -file mydomain.crt -keystore client.truststore
```

Generate a keystore and self-signed certificate (see How to Create a Self Signed Certificate using Java Keytoolfor more info)

```      
keytool -genkey -keyalg RSA -alias selfsigned -keystore server.keystore -storepass password -validity 360 -keysize 2048
```

* Java Keytool Commands for Checking

If you need to check the information within a certificate, or Java keystore, use these commands.

Check a stand-alone certificate

```
keytool -printcert -v -file mydomain.crt
```

Check which certificates are in a Java keystore

```
keytool -list -v -keystore keystore.jks
```

Check a particular keystore entry using an alias

```
keytool -list -v -keystore keystore.jks -alias mydomain
```

* Other Java Keytool Commands

Delete a certificate from a Java Keytool keystore

```
keytool -delete -alias mydomain -keystore keystore.jks
```
Change a Java keystore password

```
keytool -storepasswd -new new_storepass -keystore keystore.jks
```

Export a certificate from a keystore

```
keytool -export -alias mydomain -file mydomain.crt -keystore keystore.jks
```

List Trusted CA Certs

```
keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts
```

Import New CA into Trusted Certs

```
keytool -import -trustcacerts -file /path/to/ca/ca.pem -alias CA_ALIAS -keystore $JAVA_HOME/jre/lib/security/cacerts
```

If you need to move a certificate from Java Keytool to Apache or another type of system, check out these instructions for converting a Java Keytool keystore using OpenSSL. For more information, check out the Java Keytool documentation or check out our Tomcat SSL Installation Instructions which use Java Keytool.


### server keystore and truststore

Once the server keystore and truststore are ready, copy them to src/main/resources/config folder or externalized config folder. At the same time, update server.yml and secret.yml for keystoreName, truststoreName, keystorePass, keyPass and truststorePass. Please note that the password in secret.yml can be [encrypted][]. 

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort: 8080

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort: 8443

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled. When Http2 is enable, enableHttps is true and enableHttp is false by default.
# If you want to have http enabled, enableHttp2 must be false.
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.petstore-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false
```

And update secret.yml for passwords

```
# This file contains all the secrets for the server in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be
# mapped to Secrets and all other config files will be mapped to mapConfig

---
# Key store password, the path of keystore is defined in server.yml
keystorePass: secret

# Key password, the key is in keystore
keyPass: secret

# Trust store password, the path of truststore is defined in server.yml
truststorePass: password

# Client secret for OAuth2 server
clientSecret: f6h1FTI8Q3-7UScPZDzfXA
```

### client keystore and truststore

For one-way TLS, only client.truststore is used and you need to import the server certificate into it. For two-way TLS, both client.keystore and client.truststore will be used. 

Here is an example TLS section of the client.yml config. For the same encryption reason, the password for client keystore and private key, as well as password for client truststore, will be in the secret.yml file.

```
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: client.keystore

```


[encrypted]: /tutorial/security/encrypt-decrypt/
[keystore truststore]: /tutorial/security/keystore-truststore/
