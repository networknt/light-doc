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

A decryptor is used to inject into the framework to decrypt the encrypted values in a config file. To use encrypted values in a config file, you need a command line [utility][] to convert plain text to encrypted text and put it into the value field in your config file. This module contains several reference implementations that implement the Decryptor interface defined in the decryptor module in the light-4j repository.

The light-4j framework has many modules, and each module might have its config file that contains sensitive information. Some of our customers have a very strict policy that clear text passwords or secrets are not allowed in the configuration files. So we have to find a way to encrypt the passwords, client-secret etc. And in the framework, we need to decrypt the value back to clear text so the application can use it. 

The reference decryptor contains two parts and uses AES with salt for encryption and decryption.

We recommend that each organization implement its customized encryptor and decryptor for maximum security. However, if you don't want the extra work, you can use our default implementation of [AutoAESSaltEncryptor][] configured in the [config.yml][] file in the config module. 

If you have your own implementation class, you need to externalize the config.xml and replace the decryptorClass. Note that you cannot overwrite the values from values.yml as this config.yml is loaded at the very beginning during the server startup. 

### Encrypt Utility

If the default [AutoAESSaltEncryptor][] is used, you can use the [light-encryptor][] command line utility to do the encryption. The README.md has all the information for usage. 

### Decrypt Class

This is the other side of the coin, and this class should be included in your built API or service. The decryptor will only be invoked if values in config files are encrypted by the above utility. 

Start from light-4j 2.* version, the decryptor instance defined in the config.yml file:

```
decryptorClass: com.networknt.decrypt.AutoAESSaltDecryptor
```

User can externalize the config.yml to use customized implementation class.

### Service API Decryption process

Generally, the encryption key is the only plain text value that needs to be remembered. And the key will be saved in an environment variable or passed to the service via the java command line -D option. 

For framework default implementation decryptor(AutoAESSaltDecryptor), it uses a default environment variable name:

```
LIGHT_4J_CONFIG_PASSWORD = "light_4j_config_password";
String passwordStr = System.getenv(LIGHT_4J_CONFIG_PASSWORD);
```

As different operating systems have different conventions on the environment variable name, you can use the variable lower case and upper case to load the password. Also, once the password is loaded during the server startup, it will be cached for the entire server session. 

We have many test cases with encrypted secrets in the test configuration.  To make sure they are all working without adding an environment for each developer, we have a default password, "light," as the fallback plan if the thread started from Junit. 

If users need to use another environment variable name (for example, the environment variable for the key already existing for other systems), it can be specified in the config file:

```
DecrptKeyEnvironmentVarName: ENV_ENCRYPT_KEY
```

So the API deployment process can be implemented as following steps:

- User the encryption key to encrypt the password(s) or secret(s)
- Create a Kubernetes Sealed Secret for the encryption key if the service is deployed to a K8s cluster. 
- Put the key in the .profile or .bashrc if the service is deployed to a Linux VM. 
- From the Kubernetes deployment (or deployment config) YAML, set the environment variable and point value to the Sealed Secret created above. 

For example:
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
[utility]: https://github.com/networknt/light-encryptor
[AutoAESSaltEncryptor]: https://github.com/networknt/light-4j/blob/master/decryptor/src/main/java/com/networknt/decrypt/AutoAESDecryptor.java
[config.yml]: https://github.com/networknt/light-4j/blob/master/config/src/main/resources/config/config.yml
[light-encryptor]: https://github.com/networknt/light-encryptor


