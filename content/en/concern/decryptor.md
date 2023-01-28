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
reviewed: true
---

This module contains a reference implementation of the decryptor that implements Decryptor interface defined in the utility module. A decryptor is used to inject into the framework to decrypt the encrypted values in the secret.yml file. In order to use encrypted values in secret.yml, you also need a command line utility to convert plain text to encrypted text and put it into the value field in secret.yml

While light-4j framework is following security first design, we have grouped all sensitive config info into a centralized secret.yml that is shared for both the client and server. Some of our customers have a very strict policy that clear text passwords are not allowed in the configuration files. So we have to find a way to encrypt the passwords, client-secret etc. And in the framework, we need to decrypt the value back to clear text so that the application can use it.

The reference decryptor contains two parts and uses AES for encryption and decryption.

### Encrypt Utility

This is a command line utility that takes a string as input and outputs the encrypted text. You can pick up any value in secret.yml, encrypt it and put the result into secret.yml

If you have your implementation, you shouldn't package it into your framework or your API. This
should be a standalone utility used by the operation team or an internal website that you can 
encrypt input to output. 

### Decrypt Class

This is the other side of the coin and this class should be included into your built API or
service. The decryptor will only be invoked if values in secret.yml are encrypted by the above
utility.  With SingletonServiceFactory, you can inject any customized implementation you want.

Start from light-4j 2.* version, the decryptor instance defined in the Config.yml file:

```
decryptorClass: com.networknt.decrypt.AutoAESDecryptor
```

User can change the config to use customized implementation class.

### Service API Decryption process

In general case, the encrypt key is only plain text value need be keep and remember. And the key will be saved in an environment variable.
For framework default implementation decryptor(AutoAESDecryptor), it use default environment variable name:

```
LIGHT_4J_CONFIG_PASSWORD = "light_4j_config_password";
String passwordStr = System.getenv(LIGHT_4J_CONFIG_PASSWORD);
```
If user need use other environment variable name (for example, the environment variable for the key already existing for other systems), it can be specified in the config file:

```
#DecrptKeyEnvironmentVarName: ENV_ENCRYPT_KEY
```

So the API deployment process can be implemented as following steps:

- User the encryption key to encrypt the password(s)
- Create a kubernetes  secret for the encryption key
- From the kubernetes deploynment (or deploymentConfig) yaml, set the environment variable and point value to the secret created above
  
  
  for example:
  ```
       env:
       - name: ENV_ENCRYPT_KEY
         valueFrom:
           secretKeyRef:
             name: encryptkey
             key: keycode
```
- Add the encrypted password(s) for password value(s) in configMap 



There is a tutorial that documents the details of [how to use customized encryptor][].

 
[how to use customized encryptor]: /tutorial/security/encrypt-decrypt/
 

