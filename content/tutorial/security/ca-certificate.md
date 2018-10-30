---
title: "Convert CA-signed certificate to server.keystore"
date: 2018-10-30T13:38:56-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

A lot of our users are using Let's Encrypt to get free certificates to install on their public-facing light-4j instances. i.e., light-router as an external access point or BFF for single page applications. In the [let's encrypt][] tutorial, we have shown you how to get the certificate and key with certbot. In this tutorial, we are going to walk through the step to convert the full chain certificate and the key to a server.keystore file. 

### Cert and Key

Once completed the command line of certbot successfully, there would be two files written to your default cert directory. 

Your certificate and chain have been saved at:

```   
   /etc/letsencrypt/live/faucet.taiji.io/fullchain.pem
```
Your key file has been saved at:

```
   /etc/letsencrypt/live/faucet.taiji.io/privkey.pem
```

### Keystore

Self-signed keystore can be easily created with keytool command. But if you have a private key and a CA-signed certificate of it, you can not create a key store with just one keytool command.

Here are the steps to follow.

##### PKCS12 keystore

Create PKCS 12 file using your private key and CA signed certificate of it. You can use openssl command for this.

```
openssl pkcs12 -export -in [path to certificate] -inkey [path to private key] -certfile [path to certificate ] -out server.p12
```

As an example, assume that I have a private key called "privkey.pem" and full chain certificate called "fullchain.pem" in the current folder.

```
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -certfile fullchain.pem -out server.p12
```

It will ask you to enter the export password. Once it is done, you can find the server.p12 in the same folder.


##### Create server.keystore

Now let's create server.keystore from server.p12 generated above.

```
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -destkeystore server.keystore -deststoretype JKS
```

Created server.p12 PKCS 12 file has been given as the source keystore and new file name server.keystore has been given as the destination keystore.

As an example. 

```
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -destkeystore server.keystore -deststoretype JKS
Importing keystore server.p12 to server.keystore...
Enter destination keystore password:  
Re-enter new password: 
Enter source keystore password:  
Entry for alias 1 successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
```

For demo purpose, I am using `password` as password for both keystore and private key. 

By using the following command line you can change the private key password. 

```
keytool -keypasswd -alias [Alias name for private key] -keystore [path to key store]
```

Also, you can change the alias name for your private key.

```
keytool -changealias -keystore [path to key store] -alias [current alias]
```



[let's encrypt]: /tutorial/security/lets-encrypt/


