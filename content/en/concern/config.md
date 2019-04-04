---
title: "Config"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
---

Config is a configuration module that supports externalized config in official standalone deployment or docker container. It is encouraged that every module should have its own configuration file and these files can be served by light-config-server which aggregates/merges config files from a different level of organizations in GitHub or other git servers.

## Introduction

Externalized configuration from the application package is very important. It allows us to deploy the same package to DEV/SIT/UAT/PROD environment with different configuration packages without reopening the delivery package. For development, embedded configuration gives us the flexibility to take default values out of the box.

When dockerizing our application, the external configuration can be mapped to a volume which can be updated and restarted the container to take effect.

When Kubernetes is used for docker orchestration, config files can be managed by ConfigMap and Secret.

The reason we encourage every module has its own configuration file is to ease the management and facilitate individual module independence. Every module can be upgraded independently and change its config format without impact other modules. 

Most config files are not sensitive and it is recommended to manage them in git so that all the changes can be tracked and audited. With proper access control to the organization/lob level of config repo, only certain users can update the config files per environment. 

There is one separate config file secret.yml that contains all the secrets in the application like database password, private key etc. Although this file can be checked into the git along with other config files, the content should be just placeholders and it needs to be updated during deployment. With Kubernets, this file will be mapped to Secrets.

For now, only file system optionally associate with environment variables configuration is supported; however, config files can be served by light-config-server in a zip format and the server module can be started to load all config from light-config-server in a zip file format. It will automatically unzip it and put the config files into the right location to bootstrap the server. Given the risk of single point failure, only when the server start, it will contact external light-config-sever to load (first time) or update the local file system so that config files are cached. Except first time start the server, the server can use the local cache to start if config service is not available.

In 1.5.26 release, an external config file 'values' with extension json, yaml or yml are provided which can be optionally used to inject value into the config. By configuring this file, environment-specific values in the configuration file can be managed more easily and directly. The guidance also can be found in this document.


## Singleton

For one JVM, there should be only one instance of Config, and it should be shared by all the modules within the application. The class is designed as a singleton class and has several abstract methods that should be implemented in subclasses. A default File System based implementation is provided. Other implementation will be provided in the future when it is needed.
 
## Interface

The following abstract methods are implemented in the default service provider.

```
    public abstract Map<String, Object> getJsonMapConfig(String configName);

    public abstract Map<String, Object> getJsonMapConfigNoCache(String configName);

    public abstract Object getJsonObjectConfig(String configName, Class clazz);

    public abstract String getStringFromFile(String filename);

    public abstract InputStream getInputStreamFromFile(String filename);

    public abstract ObjectMapper getMapper();

    public abstract Yaml getYaml();

    public abstract void clear();

```

## Usage

It is encouraged to create a POJO for your config if the JSON structure is static. However, some of the config files have repeatable elements, a map or list is the only options to represent the data in this case.

The two methods return String and InputStream is for loading files such as text file with different formats and X509 certificates etc.

getMapper() is a convenient way to get Jackson ObjectMapper instead of creating one which is too heavy.


## Cache

Loading configuration files from file system takes time so the config file will be cached once it is loaded the first time. All subsequent access to the same config file (maybe different key/value pair) will read from the cached copy. In order to make sure that server startup won't be impacted by all modules to load config files, lazy loading is encouraged unless the module must be initialized during server startup.

The server [info][] module will output the config files in map format for each enabled module, and it creates another un-cached copy with getJsonMapConfigNoCache as some of the configs contains sensitive info that needs to be masked. For example, secret.yml.

## Loading sequence

Each module should have a config file that named the same as the module name. And each application or service built on top of the light-4j framework should have a config file named with application or service name if necessary. 

For example, the Security module as a config file named security.yml and it resides in the module /src/main/resources/config folder by default.

For an application that uses the Security module, it can override the default module level config by providing security.yml in application's resource folder /src/main/resources/config.

Config will also be loaded from the classpath of the application, and this is mainly used for testing as it is easy to inject different combinations of config copies in test cases into classpath for unit testing.

For official runtime environment, config files should be externalized to a separate folder outside of the package by a system property
"light-4j-config-dir". You can start the server with a "-Dlight-4j-config-dir=/config" option and put all your files into /config. Once in docker image, it can be mapped to a host volume with -v /etc/light-4j:/config in docker run command.

When Kubernetes cluster is your deployment environment, and you have config files scattered in several folders on host, they must be mounted relative to the same config folder specified by the -Dlight-4j-config-dir and all references to other files in the config file must use relative path. Here is an example. 

```
  volumeMounts:
    - name: light-config
      mountPath: /etc/research-service
    - name: google-credentials
      mountPath: /etc/research-service/gcloud
    - name: ssl-certificates
      mountPath: /etc/research-service/ssl
```

And in the server.yml file, the keystore would be something like. 

```
  keystoreName: ssl/server.keystore
  truststoreName: ssl/server.truststore
```

Given the above explanation, the loading sequence is:

1. System property "light-4j-config-dir" specified directory 
2. Class Path
3. Application resources/config folder
4. Module resources/config folder

Most modules will have a default config file with `yml` format, and it can be overwritten by putting a `yml` config in application/service resources/config folder. For unit test and end-to-end test, you can create another copy of `yml` config file under test/resources/config folder. If you need to test the different behaviors under different configuration, you need to manually manipulate the config file in the classpath in your test case in order to simulate different configurations. 

For official deploy environment, externalized config in light-4j-config-dir is recommended; however, not all config files need to be externalized. Only the ones that you might change from one environment to another or might be changed on production under certain conditions. 

The above loading sequence assumes that `yml` is used as the config file extension. If `yaml` or `json` is used, the loading sequence is the same. Remember you only use one extension for a module and these extensions cannot be mixed. 

The above loading sequence is for one extension and one extension only. For different extensions, the loading sequence is:

1. yml
2. yaml
3. json


## Format

The config module supports two file formats and three file extensions.

* YAML with extension yml and yaml
* JSON with extension json

YAML is highly recommended because it is easy to read/edit with comments inline

For each module, you can choose either YAML or JSON as config file format, but you cannot use both as there is a priority in loading sequence: 

yml > yaml > json

Note that all default config files for light-4j and other framework built on top of light-4j modules are using `yml` as config format except swagger.json which is generated from the swagger editor. If you want to overwrite the default config for one of the modules, you must use the {module name}.yml in your app config folder or externalize to config directory specified by system properties.

## Environment/External config Injection

In 1.5.26 release, values in the configuration file can be managed/override by setting environment variables or using an external config 'values' with extension `yml`, `yaml`, `json`. 

In each config file (take `yaml` as the example here),  you can replace fixed configuration as following:

```
some_key: ${VAR} 
```

Retrieve the value named 'VAR' from the environment or external config 'values. (yaml/yml.json)' and replace '${VAR}' with the value. If the value cannot be found, a ConfigException will be thrown.

```
some_key: ${VAR:default} 
```

Retrieve the value named 'VAR' from the environment or external config 'values. (yaml/yml.json)' and replace '${VAR}' with the value. If the value cannot be found,'${VAR}' will be replaced with 'default'.

```
some_key: ${VAR:?error}
```

Retrieve the value named 'VAR' from the environment or external config 'values. (yaml/yml.json)' and replace '${VAR}' with the value. If the value cannot be found, a ConfigException contains 'error' will be thrown.

```
some_key: ${VAR:$}
```  

Skip injection and '${VAR}' remains in the config.

Three injection mode are provided. If you have a privileged requirement, you can do this through the command line `java -Dinjection_order=" mode number" application_launcher_class`. Otherwise, the injection mode defaults to mode [2]. 
   
Mode [0] Inject from "values.yaml" only.
   
Mode [1] Inject from system environment first, then overwrite by "values.yaml" if exist.
   
Mode [2] Inject from "values.yaml" first, then overwrite with the values in the system environment.

After setting the injection mode, you can set environment or values. (yml/yaml/json) based on the mode you chose. environment variables can be simply export by using a command line like this. `export DOCKER_HOST_IP=192.168.1.120` while the values.(yml/yaml/json) can be set as following example. The values can be set as String, List or Map inside this external config.

whitelists.yaml:
```
paths:${IPS}
```
values.yaml:
```
IPS:
- '127.0.0.1'
- '10.10.*.*'
```
result of whitelists.yaml:
```
paths:
- '127.0.0.1'
- '10.10.*.*'
```

In version 1.5.26, all values from environment variables and default values are strings. It may cause the exception in
terms of the type casting error if do not pay attention to use, especially for setting boolean. So in 1.5.29, the type
of environment variables and default values will be cast based on their content. The rules are similar to the conversion 
rules of snakeyaml, as follows:

| Config | Result |
| ------ | ------ |
| key: ${value:1} | key set to Integer 1 |
| key: ${value:1.1} | key set to Double 1.1 |
| key: ${value:true} | key set to Boolean true |

The full list of strings that can be cast to boolean:

true: {"y", "Y", "yes", "Yes", "YES", "true", "True", "TRUE", "on", "On", "ON"}

false: {"n", "N", "no", "No", "NO", "false", "False", "FALSE", "off", "Off", "OFF"}

To further explain, a detailed example is provided below as reference:
    
server.yml:
```
buildNumber: ${server.buildNumber:latest}
```
values.yml:
```
server.buildNumber: 123
```
environment variable:
```
server.buildNumber=456
```

* case1: Both value sources exist

    If the order code equals to [0]: buildNumber: 123

    If the order code equals to [1]: buildNumber: 123

    If the order code equals to [2]: buildNumber: 456

    If the order code equals not set:buildNumber: 456 (default[2])


* case2: Only environment variable is provided

    If the order code equals to [0]: buildNumber: latest

    If the order code equals to [1]: buildNumber: 456

    If the order code equals to [2]: buildNumber: 456

    If the order code equals not set:buildNumber: 456 (default[2])


* case3: only values.yaml is provided

    If the order code equals to [0]: buildNumber: 123

    If the order code equals to [1]: buildNumber: 123

    If the order code equals to [2]: buildNumber: 123

    If the order code equals not set:buildNumber: 123 (default[2])


* case4: Neither environment variable or values.yaml is not provided

    If the order code equals to [0]: buildNumber: latest

    If the order code equals to [1]: buildNumber: latest

    If the order code equals to [2]: buildNumber: latest

    If the order code equals not set:buildNumber: latest (default[2])
    
Notes:
1. An alternate format is provided to enhance readability. Spaces can be added after the colon in curly 
braces. For example,`"{value: 1}"`, It should be noticed that the quotes outside the parentheses cannot 
be omitted in this case.

2. In order to prevent some unchangeable files from being affected by this feature. Variable injection 
can be prevented by setting config.yml. In 1.5.29, a default config.yml will be generated by light-codegen. 
For details, see the comments in the file.

## Example

Load config into static variable

```
static final Map<String, Object> config = Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
```

Load config when it is used

```
ValidatorConfig config = (ValidatorConfig)Config.getInstance().getJsonObjectConfig(CONFIG_NAME, ValidatorConfig.class);
```

## Config Server

With every module or application has its own config file, it is hard to manage these files for the different version of the framework, different deployment environment.

In order to make it easier, we have provided a [light-config-server](https://github.com/networknt/light-config-server) to manage config files in multiple git organizations. The default config files are provided by the networknt organization in GitHub. For companies to use the framework, they can have an overwritten default config repo for each version of the framework for each environment. Also, each individual application or service can have its overwritten config files for a framework version of a specific environment.

For more information about light-config-server please check [README.md](https://github.com/networknt/light-config-server)

## Handle encrypted values in config files
Encrypted values are inevitable in configuration files. It's not acceptable to put confidential information, such as passwords, access tokens, or keys, in plain text in configration files.
In early versions of light-4j (before 1.5.32), users needs to put all encrypted values in the special config file `secret.yml` and use the utility class `DecryptUtil` to decrypt the file before they can use the configured values. This approach is not convinient in two aspects:
1. it increases the difficulty in maintaining configuration files. People may forget to update or delete an encrypted configuration item when it's context changed.
2. extra efforts are needed to compose or load configurations from multiple configuration files. The extra efforts can be large when integration with third party applications is involved.

To address this issue, we enhanced the `config` module in release light-4j-1.6.0 and provide transparent access to encrypted values. With this enhancement, all values returned from `Config.getJson...()` methods are decrypted using the provided decryptor. Users can specify the decryptor in `config.yml` as shown below. There are three types of decryptor to choose from AESDcryptor, AutoAESDcryptor, and ManualAESDecryptor. The value of `decryptorClass` needs to be a concrete class that implements the interface `com.networknt.decrypt.Decryptor`. If `decryptorClass` is not configured, values are not decrypted (that is, the same behavior as before). 

```
decryptorClass: 
    com.networknt.decrypt.AESDecryptor
 #  com.networknt.decrypt.ManualAESDecryptor
 #  com.networknt.decrypt.AutoAESDecryptor
```
1. AESDecryptor: The decryptor with default password `light`. The user does not need to provide a password.  
2. ManualAESDecryptor: The decryptor that gets the password from the terminal input. The user need to input the password in the console when the server starts. The following line will be displayed in the console:
```
Password for config decryption: 
```
3. AutoAESDecryptor: The decryptor that gets the password from the environment automatically. The user needs to set the password as an environment variable by using the key `light-4j-config-password` as shown below.
```
export light-4j-config-password=your_password
```

[info]: /concern/info/
