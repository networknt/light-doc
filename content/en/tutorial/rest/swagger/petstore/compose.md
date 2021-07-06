---
title: "Compose"
date: 2018-03-10T12:30:09-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, an individual docker container is started with all the config files
default in the docker container. In this tutorial, we are going to create a docker-compose.yml
file with externalized config files so that we can change the petstore behavior with the same
container image. 

The externalized config file will be put into light-config-test repository and it contains
most config files for vary projects in testing mode.

In this step, we are going to externalize security.yml so that we can turn on or off security.
Also, we are going to externalize logback.xml to control the debugging level for the container.

The config folder and docker-compose.yml can be found at the following location. 

```
cd ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/security
``` 

First let's externalize the security.yml file to enable and disable security for the docker
container. 

Here is the default security.yml in the security/config folder. 

```yaml
# Security configuration in light framework.
---
# Enable JWT verification flag.
enableVerifyJwt: false

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: true

# User for test only. should be always be false on official environment.
enableMockJwt: false

# JWT signature public certificates. kid and certificate path mappings.
jwt:
  certificate:
    '100': oauth/primary.crt
    '101': oauth/secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false

# If OAuth2 provider support http2 protocol. If using light-oauth2, set this to true.
oauthHttp2Support: true

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: false

```

As you can see that the security is turned off. 

Let's start the docker-compose with the following command lines.

```
cd ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/security
docker-compose up
```

As security is disabled, we can use the following curl command to access the server. 

```
curl -k https://localhost:8443/v2/pet/111
```

Once you get the correct response, let's change the security.yml file to make
enableVerifyJwt to true. First let's shutdown the docker-compose with CTRL+C 

The file that is modified located at ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/security/config

The updated security.yml will have this line instead. 

```yaml
enableVerifyJwt: true
```

Now let's restart the docker-compose. 

```
cd ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/security
docker-compose up
```

If you issue the same curl command, you will receive an error. 

```
curl -k https://localhost:8443/v2/pet/111
```

There is a file called logback.xml in the same folder like security.yml and you can
change this file to enable different level of debugging for the petstore service running
inside the container. 

In the file you have a section hat controls log level for com.networknt package. 

```xml
    <logger name="com.networknt" level="error">
        <appender-ref ref="log"/>
    </logger>
``` 

Let's shutdown the docker-compose and change the level="error" to level="debug" in above
section in logback.xml and restart the docker-compose. 

What can you see? A lot more debug info? This is the way that we can enable different
logging level the application with externalized logback.xml file. 


In this tutorial, we have externalize the config files and start server with docker-compose
with externalized config files. You can put all config files into the externalized config 
folder or just put the files that you want to override the default one inside the container. 

In the next tutorial, let's enable centralized [logging][] as that is is very important for
microservices architecture. 


[logging]: /tutorial/rest/swagger/petstore/logging/

