---
title: "Regex Replace"
date: 2019-03-27T14:05:12-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This is the task that is used to do the regular expression replacement across multiple repositories. We are using this to change the dependencies version for hundreds of projects within the networknt GitHub origanization. 

Here are the steps for the task. 

* check out repositories to a workspace and switch to the specified branch. If the repsoitory has been checked already, swith to the specified branch and pull the latest code from the remote. 
* replace with regular expression pattern
* check in all repositories to the remote server

For live configuration example, please visit [light-config-test][]. 

Here is an example of regex-replace.yml for the master branch. 

```
# Workspace that is used for this operation. Most of time, this is done on local user home.
workspace: regexreplace_1_6_x
# git commit comment for the current task
comment: 'upgrade json-schema-validator to 1.0.4'
# indicate if you want to skip checkout. yes if you know that all repositories are just checkout
skip_checkout: false
# indicate if you want to skip replace. yes if you have just run the replace first and this time you just want to checkin.
skip_replace: false
# indicate if you want to skip checkin. If you are not comfortable to checkin directly, skip this step and then you can
# skip checkout and replace in second round and only make skip checkin false.
skip_checkin: false
# clone and switch to develop branch or checkout and pull from develop branch. This is to ensure that develop branch for
# each repository is up-to-date for manipulations. Please update skip_checkout to true if you want to bypass this step.
checkout:
  - branch: master
    repository:
    # repositories from github.com
    - git@github.com:networknt/light-4j.git
    - git@github.com:networknt/json-overlay.git
    - git@github.com:networknt/openapi-parser.git
    - git@github.com:networknt/light-rest-4j.git
    - git@github.com:networknt/light-graphql-4j.git
    - git@github.com:networknt/light-hybrid-4j.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-kafka.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-saga-4j.git
    - git@github.com:networknt/light-session-4j.git
    - git@github.com:networknt/light-spa-4j.git
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-tokenization.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-example-4j.git
    - git@github.com:networknt/light-docker.git
    - git@github.com:networknt/light-doc.git
    - git@github.com:networknt/light-config-test.git
    - git@github.com:networknt/light-bot.git
    - git@github.com:lightapi/light-config-server.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/model-config.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/react-schema-form.git
    - git@github.com:networknt/light-workflow-4j.git
    - git@github.com:networknt/swagger-bundler.git
    - git@github.com:networknt/openapi-bundler.git
    - git@github.com:networknt/http2client-benchmark.git
    - git@github.com:networknt/json-schema-validator.git
    - git@github.com:networknt/json-schema-validator-perftest.git
    - git@github.com:networknt/microbenchmark.git
    - git@github.com:networknt/react-schema-form-rc-select.git
    - git@github.com:networknt/light-config-prod.git
    - git@github.com:networknt/react-file-manager.git
    - git@github.com:networknt/light-commerce.git
    - git@github.com:networknt/light-cms.git
    - git@github.com:networknt/light-consumer-4j.git
    - git@github.com:networknt/http2client-benchmark.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/light-spring-boot.git

    # repositories from lightapi.net
    # - git@198.55.49.186:lightapi/tokenization.git

# regex replacement from old value to new value during a full text search on certain patterns
# of file names.
replace:
# matched path based on Glob (https://javapapers.com/java/glob-with-java-nio/)
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.10
#   new_value: 0.1.15

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.jackson>\d*\.\d*\.\d*</version.jackson>
#   old_value: 2.9.4
#   new_value: 2.9.5

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.swagger>\d*\.\d*\.\d*</version.swagger>
#   old_value: <version.swagger>1.0.32</version.swagger>
#   new_value: <version.swagger-parser>1.0.34</version.swagger-parser>

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version>[$]\{version.swagger\}</version>
#   old_value: version.swagger}
#   new_value: version.swagger-parser}

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.15
#   new_value: 0.1.18

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.commons.codec>\d*\.\d*</version.commons.codec>
#   old_value: "1.10"
#   new_value: "1.11"

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.commons-text>\d*\.\d*</version.commons-text>
#   old_value: "1.1"
#   new_value: "1.3"


# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.jose4j>\d*\.\d*\.\d*</version.jose4j>
#   old_value: "0.6.1"
#   new_value: "0.6.3"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.jose4j>\d*\.\d*\.\d*</version.jose4j>
#   old_value: "0.5.2"
#   new_value: "0.6.3"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.jose4j>\d*\.\d*\.\d*</version.jose4j>
#   old_value: "0.5.5"
#   new_value: "0.6.3"


# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.0.Final"
#   new_value: "1.4.23.Final"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.10.Final"
#   new_value: "1.4.23.Final"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.20.Final"
#   new_value: "1.4.23.Final"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.11.Final"
#   new_value: "1.4.23.Final"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.18.Final"
#   new_value: "1.4.23.Final"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "1.4.19.Final"
#   new_value: "1.4.23.Final"


# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.snakeyaml>\d*\.\d*</version.snakeyaml>
#   old_value: "1.18"
#   new_value: "1.20"

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.caffeine>\d*\.\d*\.\d*</version.caffeine>
#   old_value: "2.5.6"
#   new_value: "2.6.2"

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.javamail>\d*\.\d*\.\d*</version.javamail>
#   old_value: "1.6.0"
#   new_value: "1.6.1"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.javamail>\d*\.\d*\.\d*</version.javamail>
#   old_value: "1.4.7"
#   new_value: "1.6.1"


# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.fastscanner>\d*\.\d*\.\d*</version.fastscanner>
#   old_value: "2.7.0"
#   new_value: "2.18.1"
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.fastscanner>\d*\.\d*\.\d*</version.fastscanner>
#   old_value: "2.0.8"
#   new_value: "2.18.1"

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.hikaricp>\d*\.\d*\.\d*</version.hikaricp>
#   old_value: "3.0.0"
#   new_value: "3.1.0"

# - glob: "{**/service.yml,**/service.yml.rocker.raw}"
#   match: com.networknt.server.HandlerProvider
#   old_value: "server"
#   new_value: "handler"

# - glob: "{**/*.java}"
#   match: com.networknt.server.HandlerProvider
#   old_value: "server"
#   new_value: "handler"

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.22
#   new_value: 0.1.23

# - glob: "{**/*.java,**/*.rocker.raw}"
#   match: Http2Client.POOL,
#   old_value: POOL
#   new_value: BUFFER_POOL

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.25
#   new_value: 0.1.26

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.jackson>\d*\.\d*\.\d*</version.jackson>
#   old_value: 2.9.5
#   new_value: 2.9.8

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.26
#   new_value: 1.0.1

# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.undertow>\d*\.\d*\.\d*\.Final</version.undertow>
#   old_value: "2.0.11.Final"
#   new_value: "2.0.16.Final"

- glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
  match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
  old_value: 1.0.3
  new_value: 1.0.4
```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/regex-replace
