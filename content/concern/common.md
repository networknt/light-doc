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
---

This is the module that contains some classes that are shared between client and server. Currently
only SecretConfig is packaged into this module but there might be more in the future. 


### SecretConfig

The class SecretConfig is an object that maps to secret.yml which contains all the passwords, client
secrets etc. Basically, all sensitive information regardless used by client or server, need to be
put into this file so that it can be centralized and managed properly. 

Also this class support decryption of the sensitive values from secret.yml file if the value start
with "CRYPT". 

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
 
Please note that the reference implementation is not supposed to be used on production but merely
and example to show you how to implement a decryptor and how to create an encryption command line
utility. Further it shows how to config service.yml to inject your implementation into the framework.


To learn how to use the decryptor with secret.yml, you can follow this [encrypt/decrypt tutorial][] 

[decryptor]: /concern/descryptor/
[encrypt/decrypt tutorial]: /tutorial/security/encrypt-decrypt/
