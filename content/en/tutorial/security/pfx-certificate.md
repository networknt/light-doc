---
title: "Pfx Certificate"
date: 2021-07-15T21:29:53-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When we require a new CA certificate for the API platform, the security team will sometimes provide a pfx file that includes CA keys and certs.

A PFX file, also known as PKCS #12, is a single, password-protected certificate archive that contains the entire certificate chain plus the matching private key. Essentially it is everything that any server will need to import a certificate and private key from a single file.

We need to convert the PFX certificate to Java keystore for API TLS authentication.


### Environment requirement:

- OpenSSL

- Java 7+


For users who are using Windows with gitbash, you might experience the openssl command hung forever. If that happens, please try to use winpty to start another bash terminal to execute the openssl command. 

```
winpty bash
```

### Detail steps:


1. Get the key from the PFX file; this key is later used for p12 keystore  (change the highlighted part)

```
openssl pkcs12 -in sample.pfx  -nocerts -out keyfromppfx.key -nodes -passin pass:yourpassword

```

2. Generate crt file for truststore:

```
openssl pkcs12 -in sample.pfx -clcerts -nokeys -out config.crt -passin pass:yourpassword

```

Now, we have a key and crt file.


3. The next step is to create a truststore. Or import the signed cert to existing truststore (for now, we work on client truststore only)

If the alias already exists and we want to replace it with a new cert, we can issue a delete keytool command first.

```

keytool -delete -alias myapi -keystore client.truststore 
keytool -import -file config.crt  -alias myapi -keystore client.truststore

```

The above command imports the crt file into a JKS truststore and sets the password. For the question: "Do you trust this certificate?" answer "yes," so it is then added in the truststore.

If we only need a truststore,  we stop here.

4. Create server side keystore

```

openssl pkcs12 -export -in config.crt -inkey keyfromppfx.key -certfile config.crt -name "configcert" -out keystore.p12

```

If you are using Java 11 or above, you don't need to do anything further. The newer version of Java can use PKCS12 format keystore instead of Java specific JKS format. 

To output the server.keystore directly, you can use server.keystore to replace the keystore.p12 in the above command line. 

5. Import the p12 to server keystore.

In some cases, p12 can be used directly as a server-side keystore. But in our API platform, we can import to the server.keystore, or generate a new jks file as server keystore.

```

keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore server.keystore -deststoretype pkcs12

```

### Note

The above steps will create a client.truststore, and a server.keystore for One-Way SSL. If the API platform uses Two-Way SSL, then simply implement the same steps above for server.truststore and client.keystore.

