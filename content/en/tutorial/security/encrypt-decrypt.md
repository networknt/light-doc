---
title: "Encrypt and Decrypt"
date: 2017-12-13T15:31:55-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to show you how to use the [light-encryptor][] command line utility to encrypt a sensitive property value with a picked master key and to decrypt an encrypted value in a config file within a project with a master key environment variable on the server. 

### Decryptor Interface

The following is the interface that is used for decryption. This class is in the [decryptor][] module of light-4j.

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

The CRYPT_PREFIX is used to indicate that text is encrypted, and you need to follow the decryption process to get clear text during the server startup. 

### Reference Implementation

To ensure that our users choose a secure algorithm for encryption and decryption, we have provided a reference implementation of AES with key size 256, 64K iterations and salt. The user will choose a master key for the encryption with the light-encryptor utility and put the master key as an environment variable named LIGHT_4J_CONFIG_PASSWORD. 

By default, the reference implementation AutoAESSaltDecryptor is configured in [config.yml][], and it is used by default. If you want to replace it with your implementation, you can externalize the config.yml with your decryptorClass. 


### Encryptor Utility

Before doing anything in the API or service, we need to get the clear texts from config files, encrypt them, and put them back into config files or values.yml file.

[light-encryptor][] is a repository on GitHub that is used to encrypt the clear text. You can follow the README.md in the repo for usage. 


Here is the apikey.yml test config file. As you can see, only some of the values have been encrypted, and some are still in clear text. 

```yaml
# ApiKey Authentication Security Configuration for light-4j
# Enable ApiKey Authentication Handler, default is false.
enabled: ${apikey.enabled:true}
# path prefix to the api key mapping. It is a list of map between the path prefix and the api key
# for apikey authentication. In the handler, it loops through the list and find the matching path
# prefix. Once found, it will check if the apikey is equal to allow the access or return an error.
# The map object has three properties: pathPrefix, headerName and apiKey. Take a look at the test
# resources/config folder for configuration examples.
pathPrefixAuths:
  - pathPrefix: /test1
    headerName: x-gateway-apikey
    apiKey: abcdefg
  # The same prefix has another apikey header and value.
  - pathPrefix: /test1
    headerName: authorization
    apiKey: xyz
  - pathPrefix: /test2
    headerName: x-apikey
    apiKey: CRYPT:3ddd6c8b9bf2afc24d1c94af1dffd518:1bf0cafb19c53e61ddeae626f8906d43
```

### Summary

The above steps show you how to use encryption and decryption to protect config files. The provided reference implementation is safe to use, as users can pick the master key.

If there is sufficient demand, we might provide encryption as a service from encrypt.networknt.com so that you don't need to manage the encryptor utility within your organization. Also, we can provide a more secure approach like PKI to asymmetric encryption and decryption. 


[light-encryptor]: https://github.com/networknt/light-encryptor
[decryptor]: https://github.com/networknt/light-4j/blob/master/decryptor/src/main/java/com/networknt/decrypt/Decryptor.java
[config.yml]: https://github.com/networknt/light-4j/blob/master/config/src/main/resources/config/config.yml

