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

A decryptor is used to inject into the framework to decrypt the encrypted values in a config file. To use encrypted values in a config file, you need a command line utility to convert plain text to encrypted text and put it into the value field in your config file. This module contains several reference implementations that implement the Decryptor interface defined in the decryptor module in the light-4j repository.

The light-4j framework has many modules, and each module might have its config file that contains sensitive information. Some of our customers have a very strict policy that clear text passwords or secrets are not allowed in the configuration files. So we have to find a way to encrypt the passwords, client-secret etc. And in the framework, we need to decrypt the value back to clear text so the application can use it. 

The reference decryptor contains two parts and uses AES for encryption and decryption.

### Encrypt Utility

This command line utility takes a string as input and outputs the encrypted text. You can pick up any value in your config file, encrypt it and put the result into it to replace the clear text.

If you have your implementation, you shouldn't package it into your framework or your API. This should be a standalone utility used by the operation team or an internal website that you can encrypt input to output. 

As we use the java command line to execute the utility, we must pass the clear text secret to the utility as a command line parameter. We need to use double quotes for the clear text password to avoid any encoding errors. 

Otherwise, we might not encode the password correctly. For example, the following password cannot be encoded correctly on Windows. 

```
tN-(kw^tQ\46}Bq
```

* If the password or secret contains a back slash with numbers following it, you have to escape it with double back slashes. 

```
tN-(kw^tQ\\46}Bq
```

* If you are running the utility on windows cmd, you cannot have '^' in your password. You have to run the command line on Linux. The '^' is a special character on the Windows command line. 

So, to be safe, let's use double quotes for the password in the command line to avoid the above issues. 

Here is an example of command line. 

```
java -jar encryptor-1.0.0.jar "tN-(kw^tQ\46}Bq"
```

There is a reference implementation in the common module test folder in the light-4j repository. All users are encouraged to have their customized implementation. 

```
https://github.com/networknt/light-4j/blob/master/common/src/test/java/com/networknt/common/AESSaltEncryptor.java
```

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

Generally, the encryption key is the only plain text value that needs to be remembered. And the key will be saved in an environment variable.

For framework default implementation decryptor(AutoAESDecryptor), it uses a default environment variable name:

```
LIGHT_4J_CONFIG_PASSWORD = "light_4j_config_password";
String passwordStr = System.getenv(LIGHT_4J_CONFIG_PASSWORD);
```

If users need to use another environment variable name (for example, the environment variable for the key already existing for other systems), it can be specified in the config file:

```
#DecrptKeyEnvironmentVarName: ENV_ENCRYPT_KEY
```

So the API deployment process can be implemented as following steps:

- User the encryption key to encrypt the password(s)
- Create a Kubernetes  secret for the encryption key
- From the Kubernetes deployment (or deployment config) YAML, set the environment variable and point value to the secret created above
  
  
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
 

