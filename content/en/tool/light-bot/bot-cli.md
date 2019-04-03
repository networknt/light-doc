---
title: "Invoke light-bot Cli"
date: 2019-03-25T14:06:24-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-bot is built as a Java application, and it has a command line that allows you to execute a task with internal or externalized config directory. The command line utility is an entry point to call all the major tasks.

To run the command line, go to `~/networknt/light-bot/bot-cli/build/libs` folder

* java -jar bot-cli-fat-1.0.jar -t develop-build 
* java -jar bot-cli-fat-1.0.jar -t version-upgrade
* java -jar bot-cli-fat-1.0.jar -t release-maven 

For the entire list of tasks, please visit [how light-bot works][].

The above commands can invoke the light-bot cli but you will use the default configuration files from the bot-cli module. These config files are just examples, and they are not supposed to be used in your project. You need to pass in your own configuration files in a config folder to direct light-bot to work with your GitHub organization and repositories. 

For most of the tasks, you will need the following config files.

* cli.yml

This config file is to control the cli module. Currently, there is only one variable in this file to allow the cli to skip email report once the task is completed. If you are running a task on a remote server with cron job, an email report is very important. 

```
# This is the config file for Cli command line
# Indicate if there is an email will be sent out if build fails.
skipEmail: true
```

* email.yml

This is the config file for the email module to allow light-bot to send out an email if necessary. 

```
# Email Sender Configuration
---
# Email server host name or IP address
host: mail.lightapi.net
# Email SMTP port number. Please don't use port 25 as it is not safe
port: 587
# Email user or sender address.
user: noreply@lightapi.net
# Enable debug. Default to false.
debug: true
# Enable Authentication. Default to true.
auth: true

```

* secret.yml

You can use this file for email password and encrypted it with the light-4j [encrypt and decrypt][]

```
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
# consulToken: the_one_ring

# EmailSender password default address is noreply@lightapi.net
emailPassword: change-to-real-password

```

With the latest version of 1.5.x and above, you can put the password into the environment variable and pass it to the light-bot during the runtime. 

* service.yml

This is a very important file that you can map an interface to the real plugin implementations. By updating this file, you can add your own task with a jar file in the classpath. You can also change the existing task to customize the behavior. 

```
# Singleton service factory configuration/IoC injection
singletons:
# Executor to CommandExecutor binding
- com.networknt.bot.core.Executor:
  - com.networknt.bot.core.CommandExecutor
- com.networknt.bot.core.Command:
  - com.networknt.bot.develop.DevelopBuildTask
  - com.networknt.bot.version.VersionUpgradeTask
  - com.networknt.bot.release.ReleaseMavenTask
  - com.networknt.bot.docker.ReleaseDockerTask
  - com.networknt.bot.regex.replace.RegexReplaceTask
  - com.networknt.bot.branch.CreateBranchTask
  - com.networknt.bot.branch.MergeBranchTask
```

* Task config file

Each task has its specific config file to control how the task will be executed. Here is an example for regex-replace.yml

```
# Workspace that is used for this operation. Most of time, this is done on local user home.
workspace: regexreplace
# indicate if you want to skip checkout. yes if you know that all repositories are just checkout
skip_checkout: false
# indicate if you want to skip replace. yes if you have just run the replace first and this time you just want to checkin.
skip_replace: false
# indicate if you want to skip checkin. If you are not comfortable to checkin directly, skip this step and then you can
# skip checkout and replace in second round and only make skip checkin false.
skip_checkin: true
# clone and switch to develop branch or checkout and pull from develop branch. This is to ensure that develop branch for
# each repository is up-to-date for manipulations. Please update skip_checkout to true if you want to bypass this step.
checkout:
  - branch: develop
    repository:
    - git@github.com:networknt/light-4j.git
    - git@github.com:networknt/openapi-parser.git
    - git@github.com:networknt/light-rest-4j.git
    - git@github.com:networknt/light-graphql-4j.git
    - git@github.com:networknt/light-hybrid-4j.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-saga-4j.git
    - git@github.com:networknt/light-session-4j.git
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-example-4j.git
    - git@github.com:networknt/light-docker.git
    - git@github.com:networknt/light-doc.git
    - git@github.com:networknt/light-bot-config.git
    - git@github.com:networknt/light-config-test.git
    - git@github.com:networknt/light-bot.git
    - git@github.com:networknt/light-config-server.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/model-config.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/react-schema-form.git
    - git@github.com:networknt/light-workflow-4j.git
    - git@github.com:networknt/light.git
    - git@github.com:networknt/swagger-bundler.git
    - git@github.com:networknt/http2client-benchmark.git
    - git@github.com:networknt/json-schema-validator-perftest.git
    - git@github.com:networknt/microbenchmark.git
    - git@github.com:networknt/react-schema-form-rc-select.git
    - git@github.com:networknt/light-config-prod.git
    - git@github.com:networknt/react-file-manager.git
    - git@github.com:networknt/light-commerce.git
    - git@github.com:networknt/light-cms.git

# regex replacement from old value to new value during a full text search on certain patterns
# of file names.
replace:
# matched path based on Glob (https://javapapers.com/java/glob-with-java-nio/)
- glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
  match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
  old_value: 0.1.10
  new_value: 0.1.15

```

For examples of the task config file, please visit [light-config-test][]

With the externalized config folder in the command line, it becomes very long. We usually create a script called run.sh to execute the command. 

Here is one of the example run.sh and you can find others in the [light-config-test][].

```
#!/bin/bash

java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -jar ~/networknt/light-bot/bot-cli/build/libs/bot-cli-fat-1.0.jar -t develop-build
```

From the above java command, you can see how to specify the config folder, log file config, location of the jar and the task is about to execute. 


[how light-bot works]: /tool/light-bot/how-it-works/
[encrypt and decrypt]: /tutorial/security/encrypt-decrypt/
[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot
