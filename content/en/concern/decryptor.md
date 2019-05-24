---
title: "Decryptor"
date: 2017-12-13T14:57:50-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This module contains a reference implementation of decryptor that implements Decryptor interface
defined in utility module. A decryptor is used to inject into the framework to decrypt the encrypted
values in secret.yml file. In order to use encrypted values in secret.yml, you also need a command
line utility to convert plain text to encrypted text and put it into the value field in secret.yml

While light-4j framework is following security first design, we have grouped all sensitive config
info into a centralized secret.yml that is shared for both client and server. Some of our customers
have very strict policy that clear text password is not allowed in the configuration files. So we
have to find a way to encrypt the passwords, client-secret etc. And in the framework, we need to
decrypt the value back to clear text so that the application can use it. 

The reference decryptor contains two parts and it is used AES for encryption an decryption.

### Encrypt Utility

This is a command line utility that takes a string as input and output the encrypted text. You can
pickup any value in secret.yml, encrypt it and put the result into secret.yml

If you have your implementation, you shouldn't package it into your framework or your API. This
should be a standalone utility used by the operation team or an internal website that you can 
encrypt input to output. 

### Decrypt Class

This is the other side of the coin and this class should be included into your built API or
service. The decryptor will only be invoked if values in secret.yml are encrypted by the above
utility. With SingletonServiceFactory, you can inject any customized implementation you want.

There is a tutorial that documents the details of [how to use customized encryptor][].

 
[how to use customized encryptor]: /tutorial/security/encrypt-decrypt/
 

