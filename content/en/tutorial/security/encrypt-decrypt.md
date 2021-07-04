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

In this tutorial, we are going to show you how to create a customized encryptor command line
utility and a customized decryptor class inside your API. It also shows you how to wire in
your decryptor in service.yml file. 

### Decryptor Interface

The following is the interface that is used for decryption. This class is in utility module.

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

The CRYPT_PREFIX is used to indicate that text is encrypted and you need to follow the decryption
process to get the clear text. 


### Encryptor Utility

Before we do anything in the API or service, we need to get the clear text from secret.yml and
encrypt it and put it back into secret.yml file.

First we create a Java utility command line. Remember that you shouldn't package your utility into
your API or service. Here is for demo purpose, I put the class into the test folder in decryptor module.

```java
package com.networknt.decrypt;

import com.networknt.utility.Constants;
import sun.misc.BASE64Encoder;

import javax.crypto.*;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
import java.security.AlgorithmParameters;
import java.security.InvalidKeyException;
import java.security.spec.InvalidParameterSpecException;
import java.security.spec.KeySpec;

import static com.networknt.utility.Decryptor.CRYPT_PREFIX;
import static java.lang.System.exit;

public class AESEncryptor {
    public static void main(String [] args) {
        if(args.length == 0) {
            System.out.println("Please provide plain text to encrypt!");
            exit(0);
        }
        AESEncryptor encryptor = new AESEncryptor();
        System.out.println(encryptor.encrypt(args[0]));
    }

    private static final int ITERATIONS = 65536;
    private static final int KEY_SIZE = 128;
    private static final byte[] SALT = { (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0 };
    private static final String STRING_ENCODING = "UTF-8";
    private SecretKeySpec secret;
    private Cipher cipher;
    private BASE64Encoder base64Encoder;

    public AESEncryptor() {
        try {
           /* Derive the key, given password and salt. */
            SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
            KeySpec spec;

            spec = new PBEKeySpec(Constants.FRAMEWORK_NAME.toCharArray(), SALT, ITERATIONS, KEY_SIZE);
            SecretKey tmp = factory.generateSecret(spec);
            secret = new SecretKeySpec(tmp.getEncoded(), "AES");

            // CBC = Cipher Block chaining
            // PKCS5Padding Indicates that the keys are padded
            cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");

            // For production use commons base64 encoder
            base64Encoder = new BASE64Encoder();
        } catch (Exception e) {
            throw new RuntimeException("Unable to initialize", e);
        }
    }

    /**
     * Encrypt given input string
     *
     * @param input
     * @return
     * @throws RuntimeException
     */
    public String encrypt(String input)
    {
        try
        {
            byte[] inputBytes = input.getBytes(STRING_ENCODING);
            // CBC = Cipher Block chaining
            // PKCS5Padding Indicates that the keys are padded
            cipher.init(Cipher.ENCRYPT_MODE, secret);
            AlgorithmParameters params = cipher.getParameters();
            byte[] iv = params.getParameterSpec(IvParameterSpec.class).getIV();
            byte[] ciphertext = cipher.doFinal(inputBytes);
            byte[] out = new byte[iv.length + ciphertext.length];
            System.arraycopy(iv, 0, out, 0, iv.length);
            System.arraycopy(ciphertext, 0, out, iv.length, ciphertext.length);
            return CRYPT_PREFIX + ":" + base64Encoder.encode(out);
        } catch (IllegalBlockSizeException e) {
            throw new RuntimeException("Unable to encrypt", e);
        } catch (BadPaddingException e) {
            throw new RuntimeException("Unable to encrypt", e);
        } catch (InvalidKeyException e) {
            throw new RuntimeException("Unable to encrypt", e);
        } catch (InvalidParameterSpecException e) {
            throw new RuntimeException("Unable to encrypt", e);
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("Unable to encrypt", e);
        }
    }
}

```

As you can see there is a main method that encrypt the input text and output the result in the console.

Let's make the input as "password" and copy the result into secret.yml

Here is the end result of secret.yml file. As you can see only some of the values have been encrypted
and some are still in clear text. 

```yaml
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=

# Key password, the key is in keystore
serverKeyPass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=

# Key password, the key is in keystore
clientKeyPass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: CRYPT:08eXg9TmK604+w06RaBlsPQbplU1F1Ez5pkBO/hNr8w=

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
# consulToken: the_one_ring

```

The secret.yml can be found in src/test/resources/config folder in decryptor module.

### Decryptor Class

Now we have partial of secret.yml encrypted already, let's create a class that can do the decryption
with the same key as encryption. The key used is Constants.FRAMEWORK_NAME which is "light" in this 
case.

Here is the AESDescryptor class. 

```java
package com.networknt.decrypt;

import com.networknt.utility.Constants;
import com.networknt.utility.Decryptor;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.spec.KeySpec;

public class AESDecryptor implements Decryptor {
    private static final int ITERATIONS = 65536;

    private static final String STRING_ENCODING = "UTF-8";

    /**
     * If we user Key size of 256 we will get java.security.InvalidKeyException:
     * Illegal key size or default parameters , Unless we configure Java
     * Cryptography Extension 128
     */
    private static final int KEY_SIZE = 128;

    private static final byte[] SALT = { (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0, (byte) 0x0 };

    private SecretKeySpec secret;

    private Cipher cipher;

    private BASE64Decoder base64Decoder;

    public AESDecryptor() {
        try
        {
            /* Derive the key, given password and salt. */
            SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
            KeySpec spec;

            spec = new PBEKeySpec(Constants.FRAMEWORK_NAME.toCharArray(), SALT, ITERATIONS, KEY_SIZE);
            SecretKey tmp = factory.generateSecret(spec);
            secret = new SecretKeySpec(tmp.getEncoded(), "AES");

            // CBC = Cipher Block chaining
            // PKCS5Padding Indicates that the keys are padded
            cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");

            // For production use commons base64 encoder
            base64Decoder = new BASE64Decoder();
        }
        catch (Exception e)
        {
            throw new RuntimeException("Unable to initialize AESDecryptor", e);
        }
    }

    public String decrypt(String input) {
        if (!input.startsWith(CRYPT_PREFIX))
        {
            throw new RuntimeException("Unable to decrypt, input string does not start with 'CRYPT'");
        }

        try
        {
            byte[] data = base64Decoder.decodeBuffer(input.substring(6, input.length()));
            int keylen = KEY_SIZE / 8;
            byte[] iv = new byte[keylen];
            System.arraycopy(data, 0, iv, 0, keylen);
            cipher.init(Cipher.DECRYPT_MODE, secret, new IvParameterSpec(iv));
            return new String(cipher.doFinal(data, keylen, data.length - keylen), STRING_ENCODING);
        }
        catch (Exception e)
        {
            throw new RuntimeException("Unable to decrypt.", e);
        }
    }
}

``` 

### Plugin the decryptor

We are going to use service.yml to bind the customized implementation to Decryptor interface. Here
is the content of service.yml file which can be found in the src/test/resources/config folder of
decryptor module.

```yaml
# singleton service factory configuration
singletons:
# HandlerProvider implementation
- com.networknt.utility.Decryptor:
  - com.networknt.decrypt.AESDecryptor
```

As you can see we have bound AESDecryptor the Decryptor interface. If you have customized implementation,
you can replace the AESDecryptor implementation with your package and class. 

### DecryptUtil 

Now let's take a look at DecryptUtil class to understand how the decryption logic is called. This class
is in common module which is shared between client and server. 

```java
package com.networknt.common;

import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.Decryptor;
import org.owasp.encoder.Encode;

import java.util.List;
import java.util.Map;

public class DecryptUtil {
    public static Map<String, Object> decryptMap(Map<String, Object> map) {
        decryptNode(map);
        return map;
    }

    private static void decryptNode(Map<String, Object> map) {
        for (Map.Entry<String, Object> entry : map.entrySet()) {
            String key = entry.getKey();
            Object value = entry.getValue();
            if (value instanceof String)
                map.put(key, decryptObject(value));
            else if (value instanceof Map)
                decryptNode((Map) value);
            else if (value instanceof List) {
                decryptList((List)value);
            }
        }
    }

    private static void decryptList(List list) {
        for (int i = 0; i < list.size(); i++) {
            if (list.get(i) instanceof String) {
                list.set(i, decryptObject((list.get(i))));
            } else if(list.get(i) instanceof Map) {
                decryptNode((Map<String, Object>)list.get(i));
            } else if(list.get(i) instanceof List) {
                decryptList((List)list.get(i));
            }
        }
    }

    private static Object decryptObject(Object object) {
        if(object instanceof String) {
            if(((String)object).startsWith(Decryptor.CRYPT_PREFIX)) {
                Decryptor decryptor = SingletonServiceFactory.getBean(Decryptor.class);
                if(decryptor == null) throw new RuntimeException("No implementation of Decryptor is defined in service.yml");
                object = decryptor.decrypt((String)object);
            }

        }
        return object;
    }
}

```

As you can see it iterates all values in a map and check if the value is encrypted with the prefix and decrypt 
it if necessary. 

### Test

Now let's create a test case to see if in the application we can get clear text "password" from
secret.yml config file. The following test case can be found in common module test folder. 

```java
package com.networknt.common;

import com.networknt.config.Config;
import org.junit.Assert;
import org.junit.Test;

import java.util.Map;

public class DecryptUtilTest {
    @Test
    public void testDecryptMap() {
        Map<String, Object> secretMap = Config.getInstance().getJsonMapConfig("secret");
        DecryptUtil.decryptMap(secretMap);
        Assert.assertEquals("password", secretMap.get("serverKeystorePass"));
    }
}

```

### Summary

The above steps show you how to customize the encryption and decryption to protect secret.yml config file. The
provided reference implementation is not safe to use as anybody can read the source code to figure out the
key that is used. 

If there is sufficient demands, we might in the further to provide encryption as a service from 
encrypt.networknt.com so that you don't need to manage the encryptor utility within your organization.
Also, we can provide even more secured approach like PKI to asymmetric encryption and decryption. 

