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

## Encrypted Configuration Values in Light-4j

To keep sensitive information secure within configuration files, Light-4j allows for encryption and decryption of values in config files. This ensures that secrets like passwords and client keys remain protected, even in plain-text configuration files. 

### Overview

A **Decryptor** component is injected into the Light-4j framework to decrypt values stored in an encrypted format in configuration files. To set up encrypted values in a config file, you’ll need to use a command-line [utility][] to convert plain text into encrypted text, then place this encrypted value in the config file.

The `light-4j` repository provides several reference implementations that conform to the **Decryptor** interface. While Light-4j includes default implementations, we recommend that each organization create its own custom encryptor and decryptor for added security.

### Why Encryption is Necessary

Light-4j has multiple modules, each potentially with its own configuration file containing sensitive data. Some customers enforce strict policies that forbid clear-text passwords or secrets in config files. To address this, passwords, client secrets, and other sensitive values can be encrypted, then decrypted by the framework at runtime, so applications can securely use them.

The reference encryptor and decryptor in Light-4j use AES encryption with salt for both encryption and decryption.

### Default Decryptor Implementation

While custom implementations are recommended for optimal security, you can use the default [AutoAESSaltEncryptor][] in Light-4j, which is configured in the `config.yml` file. 

If you choose to create a custom decryptor, externalize your `config.yml` and set `decryptorClass` to the name of your custom class. Note that `config.yml` values can’t be overwritten with `values.yml` because `config.yml` is loaded at server startup.

### Encryption Utility

To encrypt values using the default [AutoAESSaltEncryptor][], you can use the `light-encryptor` command-line utility. Instructions for usage are available in its README file.

### Decryptor Class

The decryptor class handles the decryption process within your API or service. It will only decrypt values that were previously encrypted by the `light-encryptor` utility. 

Starting from Light-4j version 2.*, the decryptor class is specified in `config.yml` as follows:

```yaml
decryptorClass: com.networknt.decrypt.AutoAESSaltDecryptor
```

You can externalize `config.yml` to use a custom decryptor implementation if needed.

### Service API Decryption Process

The encryption key is typically the only value that needs to remain in plain text, stored securely as an environment variable or passed to the service with the `-D` Java option.

For the default decryptor (`AutoAESSaltDecryptor`), the framework uses an environment variable to retrieve the encryption key:

```java
String passwordStr = System.getenv("LIGHT_4J_CONFIG_PASSWORD");
```

Different operating systems may handle environment variable names differently, so both uppercase and lowercase names are supported. Once the password is loaded at startup, it’s cached for the server’s session duration. 

To accommodate testing, Light-4j uses a default password (`"light"`) as a fallback if a test is run without the environment variable (e.g., during JUnit tests).

If you need to specify a different environment variable for the key (e.g., to avoid conflicts with existing system variables), you can define it in the config file:

```yaml
DecryptKeyEnvironmentVarName: ENV_ENCRYPT_KEY
```

### Deploying Encrypted Configuration in API Services

To deploy encrypted configuration values, follow these steps:

1. Use the encryption key to encrypt passwords or secrets.
2. If deploying to a Kubernetes (K8s) cluster, create a **Kubernetes Sealed Secret** for the encryption key.
3. If deploying to a Linux VM, store the encryption key in `.profile` or `.bashrc`.
4. Set the environment variable in the Kubernetes deployment YAML to reference the Sealed Secret, as shown below:

   ```yaml
   env:
     - name: ENV_ENCRYPT_KEY
       valueFrom:
         secretKeyRef:
           name: encryptkey
           key: keycode
   ```

5. Add the encrypted password(s) to your configuration in the `configMap`.

For more information on creating a custom encryptor, refer to our [custom encryptor tutorial][].


Here’s a revised version of your documentation for decrypting secrets into clear text:

---

### Utility for Decrypting Secrets to Clear Text

In some cases, you may need to decrypt a secret from a configuration file into clear text. While there isn't a dedicated repository for this task, a test case is available in the Light-4j repository that demonstrates how to perform it. 

The test case, `testDecryptWithEnv`, is located in `decryptor/src/test/java/com/networknt/decrypt/AESDecryptorTest.java`.

#### How to Use the Decryptor Test Case

1. **Set Up Your Secret**: 
   - Copy and paste your encrypted secret into the `secretText` variable within the test case.

2. **Set Up the Environment Variable for the Master Key**:
   - Define the environment variable for the master encryption key (`LIGHT_4J_CONFIG_PASSWORD`). This can be done within your IDE or directly in your operating system (OS).

#### Setting the Environment Variable

**In the IDE:**

   - If you're running the test case in an IDE, add the following line to the Environment Variables in the Run/Debug Configuration:

     ```
     LIGHT_4J_CONFIG_PASSWORD=MASTERKEY
     ```

**In the OS:**

   - If running from the command line, you can set the environment variable either by adding it to your `.profile` file or by defining it directly in the terminal.

   - To set it permanently in `.profile`:

     ```bash
     export LIGHT_4J_CONFIG_PASSWORD=MASTERKEY
     ```

   - After adding this line to `.profile`, run the following command to load the changes into the current terminal session:

     ```bash
     source .profile
     ```

Alternatively, you can set the environment variable temporarily in the command line session without modifying `.profile`:

   ```bash
   export LIGHT_4J_CONFIG_PASSWORD=MASTERKEY
   ```

By following these steps, you’ll be able to decrypt secrets from your configuration files into clear text using the provided test case in Light-4j.


 
[how to use customized encryptor]: /tutorial/security/encrypt-decrypt/
[utility]: https://github.com/networknt/light-encryptor
[AutoAESSaltEncryptor]: https://github.com/networknt/light-4j/blob/master/decryptor/src/main/java/com/networknt/decrypt/AutoAESDecryptor.java
[config.yml]: https://github.com/networknt/light-4j/blob/master/config/src/main/resources/config/config.yml
[light-encryptor]: https://github.com/networknt/light-encryptor


