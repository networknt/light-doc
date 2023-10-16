---
title: "Customized Decryptor"
date: 2023-10-16T14:48:38-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Some organizations that don't want to use the default implementation can use their customized encryptor and decryptor. To demo how to use a customized decryptor, we have provided a test implementation [DummyDecryptor][]. Because it doesn't do any encryption but just adds the prefix CRYPT in front of the clear text, please don't use it in any official environment. 

If you implement your decryptor in a common module, you can release it internally to the organization's Artifactory and include it in your service dependency. 

### coonfig.yml

Once you have the decryptor implemented, you should externalize config.yml into your application src/main/resources/config folder. Here is the modified config.yml from the light-4j repository. 

```
#----------------------------------------------------------------------------------------------------------------
# Scalable Config file
#
# This file serves as a configuration extension platform. Functions are list below:
#
# [1]   exclusionConfigFileList: this configuration will be used by the light-4j/config module, when reading
#       config files. it allows the listing of files which will be excluded from parameterized values set at
#       the command-line or in a values.yml file
#       Notes:
#       File name included in the list will be excluded
#       If the file is not provided, the config module will safely ignore it
#       Simply list the config file names without extensions(.json, .yaml, .yml)
#----------------------------------------------------------------------------------------------------------------
exclusionConfigFileList:
  - openapi
  - values

# The AutoAESSaltDecryptor is the most secure one we have implemented, and it can be used by
# everyone with a master key set up as an environment variable on the deployment environment.
# To encrypt your sensitive secret with a command line tool, please visit the following URL.
# https://github.com/networknt/light-encryptor
# To learn how the decryptor works, please visit the document URL.
# https://doc.networknt.com/concern/decryptor/
decryptorClass: com.networknt.decrypt.DummyDecryptor
# decryptorClass: com.networknt.decrypt.AESSaltDecryptor
# decryptorClass: com.networknt.decrypt.ManualAESSaltDecryptor

# For some configuration files, we have left some properties without default values as there
# would be negative impact on the application security. The following config will ensure that
# null will be used when the default value is empty without stopping the server during the start.
allowDefaultValueEmpty: true

```

Copy the above config.yml and change the decryptorClass to your implementation class from the DummyDecryptor. 

### Test

To test it with the [petstore][] API, we need to update the values.yml to the following.


```

#--------------------------------------------------------------------------------
# values.yml : Set of values commonly overridden in microservices
# 			   The file can be extended with other elements, as necessary 
#--------------------------------------------------------------------------------

# client.yml
# https://github.com/networknt/light-4j/blob/master/client/src/main/resources/config/client.yml
client.timeout: 3000
client.verifyHostname: true

# server.yml
# https://github.com/networknt/light-4j/blob/master/server/src/main/resources/config/server.yml
server.httpPort: 8080
server.enableHttp: false
server.httpsPort: 9443
server.enableHttps: true
server.enableHttp2: true
server.enableRegistry: false
server.serviceId: com.networknt.petstore-3.0.1
server.buildNumber: 3.0.1
server.keystorePass: CRYPT:password

# openapi-security.yml
# https://github.com/networknt/light-rest-4j/blob/master/openapi-security/src/main/resources/config/openapi-security.yml
openapi-security.enableVerifyJwt: false

# metrics.yml
metrics.enabled: false

# service.yml
service.singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: portal
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.portal.registry.client.PortalRegistryClient:
  - com.networknt.portal.registry.client.PortalRegistryClientImpl
- com.networknt.registry.Registry:
  - com.networknt.portal.registry.PortalRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  
  
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  

```

In the above values.yml, we have added the following line and it needs the DummyDecryptor to decrypt during the server startup. 

```
server.keystorePass: CRYPT:password
```

If you can start the server without any error, then it approves the decryptor works. 


[DummyDecryptor]: https://github.com/networknt/light-4j/blob/master/decryptor/src/main/java/com/networknt/decrypt/DummyDecryptor.java
[petstore]: https://github.com/networknt/light-example-4j/tree/release/rest/petstore-maven-single



