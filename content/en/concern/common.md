---
title: "Common"
date: 2017-12-13T14:57:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true 
---

This is the module that contains some classes that are shared between the client and server.


### DecryptUtil

The class is a utility to handle the encrypted sensitive values in secret.yml or other config files. Basically, when using light-4j frameworks, it is recommended to put all the secrets into one file called secret.yml so that it can be managed separately and can be mapped to Secret in the Kubernetes cluster naturally.

This class contains a static API that can iterate a map to decrypt all encrypted values and return the same map as result. It blindly does the iteration without checking if the input map is empty or not. The caller is responsible to ensure that the map is not empty as the call has the context info on which config file is loaded and give proper error message if it is missing in the deployed environment.
 
For the encrypted values in config files, the value starts with “CRYPT”.

This utility calls an implementation of Decryptor interface to resolve the encrypted values. 

There is an interface for the decryptor in utility module. 

```java
public interface Decryptor {

    String CRYPT_PREFIX = "CRYPT";

    /**
     * This is the method that decrypt an encrypted value. The logic of
     * encryption and decryption are free to be implemented by the customer.
     *
     * A default implementation is included but customers should have done
     * their own implementation so this is an open source project. Also, if
     * there are enough demand, we might be able to provide encryption as
     * service from encrypt.networknt.com and provide decryption jar file per
     * customer with is more secure.
     *
     * @param input encrypted string
     * @return decrypted string
     */
    String decrypt(String input);

}

``` 

And a reference AES implementation as well a command line utility can be found in [decryptor][]
 
Please note that the reference implementation is not supposed to be used on production but merely an example to show you how to implement a decryptor and how to create an encryption command line utility. Furthermore, it shows how to config service.yml to inject your implementation into the framework.

To learn how to use the decryptor with secret.yml, you can follow this [encrypt/decrypt tutorial][] 

The following is an example on how to use this utility in Http2Client.java

```java
    static {
        Map<String, Object> secretMap = Config.getInstance().getJsonMapConfig(CONFIG_SECRET);
        if(secretMap != null) {
            secretConfig = DecryptUtil.decryptMap(secretMap);
        } else {
            throw new ExceptionInInitializerError("Could not locate secret.yml");
        }
    }
```

[decryptor]: /concern/descryptor/
[encrypt/decrypt tutorial]: /tutorial/security/encrypt-decrypt/
