---
title: "Generate CA-signed certificate (PFX) to server.keystore and client.truststore"
date: 2021-07-30T13:38:56-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When we require a new CA certificate for API platform,  sometimes security team will provide a pfx file which include CA keys and certs.

A PFX file, also known as PKCS #12 , is a single, password protected certificate archive that contains the entire certificate chain plus the matching private key. Essentially it is everything that any server will need to import a certificate and private key from a single file

Then we will need convert the PFX certificate  to Java keystore for API authentication.



### Environment requirement:

- OpenSSL

- Java 7+




### Detail steps:



1. Get the key from the PFX file; this key is later used for p12 keystore  (change the highlighted part)

```aidl
openssl pkcs12 -in sample.pfx  -nocerts -out keyfromppfx.key -nodes -passin pass:yourpassword

```



2. Generate  crt for truststore:

```aidl
openssl pkcs12 -in sample.pfx -clcerts -nokeys -out config.crt -passin pass:yourpassword

```



Now, we have a key and crt file.

3. The next step is to create a truststore. Or import the signed cert to existing truststore (for now we work on client truststore only)

---  in case the alias already existing alias and we want to replace it with new cert, we can issue delete keytool command first.

```aidl

keytool -delete -alias myapi -keystore client.truststore     

keytool -import -file config.crt  -alias myapi -keystore client.truststore

```



As we can see here,  just import this crt file into a JKS truststore and set the password. For the question: "Do you trust this certificate?" answer "yes," so it is then added in the truststore.

If we only need a truststore,  we stop here.



4. Create server side keystore

```aidl

openssl pkcs12 -export -in config.crt -inkey keyfromppfx.key -certfile config.crt -name "configcert" -out keystore.p12

```


5. Import the p12 to server keystore.

In some cases, p12 can be used directly as server side keystore. But in our API platform, we can import to the server keystore, or generate a new jks as server keystore.

```aidl

keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore server.keystore -deststoretype pkcs12

```





### Note:

The above steps will create client truststore and server keystore for One-Way SSL. If the API platform use Two-Way SSL, then simply implement same steps above for server truststore and client keystore.