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







[let's encrypt]: /tutorial/security/lets-encrypt/


