---
title: "Regex Replace"
date: 2018-04-03T05:46:21-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Often you need to replace something in your projects across multiple repositories or even multiple organizations or git servers. The manual process would be: 

* Clone all repositories into a workspace
* Do a full-text search and replace for the workspace
* Check in all repositories

If you have several repositories, it will not take a long time to do so. However, if you have dozens or hundreds of repositories, it would be a daunting job. 

What if you have to do it on a regular basis. For example, upgrade a library to a certain version across all repositories. 

light-bot regex-replace is a task created to do the replacement automatically for us. In this tutorial, we will upgrade Jackson jackson-databind from 2.9.1 to 2.9.4 as 2.9.1 has CVE-2017-17485 deserialization flaw found by one of our customers during a security scan. 

### Build light-bot

The first step is to build light-bot locally so that we can use the cli tool. Please refer to [build light-bot][] tutorial for details.


[build light-bot]: /tutorial/bot/build-light-bot/

### Check out light-config-test

Once light-bot is built successfully, you can clone light-config-test repository to the same workspace as light-bot. This repository contains configuration files for the regex-replace task. 

Once light-bot is built successfully, you can clone light-config-test repository to the same workspace as light-bot. This repository contains configuration files for the regex-replace task and script to execute the task. 

regex-replace configuration file can be found in light-config-test/light-bot/regex-replace/replace-all/config folder. Here is the current content. 

regex-replace.yml

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
  branch: develop
  repository:
  # repositories from github.com  
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
  - git@github.com:networknt/light-portal.git
  - git@github.com:networknt/light-example-4j.git
  - git@github.com:networknt/light-docker.git
  - git@github.com:networknt/light-doc.git
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
  # repositories from lightapi.net
  - git@198.55.49.186:lightapi/tokenization.git

# regex replacement from old value to new value during a full text search on certain patterns
# of file names.
replace:
# matched path based on Glob (https://javapapers.com/java/glob-with-java-nio/)
# - glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
#   match: <version.json-schema-validator>\d*\.\d*\.\d*</version.json-schema-validator>
#   old_value: 0.1.10
#   new_value: 0.1.15

- glob: "{**/pom.xml,**/pom.xml.rocker.raw}"
  match: <version.jackson>\d*\.\d*\.\d*</version.jackson>
  old_value: 2.9.1
  new_value: 2.9.4

```

### Customize Config

In the above regex-replace.yml file, you can skip each step. For example, skip the automatic checkin so that you can verify the result and then run the task again with skip checkout and skip replace. 

You can change the repositories to your and decide which branch you want to perform the task. 

At the end of the file, you can see we have set up to replace jackson version from 2.9.1 to 2.9.4 in all pom.xml and pom.xml.rocker.raw template files in light-codegen


### Execute Task

Sometimes you might want to clean up your existing workspace so that you can clone all repositories freshly. It is not necessary but if you want to do it, use the following command. 

```
cd ~
rm -rf regexreplace
```

To execute the task:

```
./run.sh
```

After the first run, you can use the following command to check if the 2.9.1 still exists in the workspace. 

```
cd ~/regexreplace
grep -R --include="pom.xml" "2.9.1" .
```
You can also, use "git status" to check how many files are updated. 

If you are comfortable at the moment, you can update the regex-replace.yml to set skip_checkin to false and rerun the run.sh script. This time, all the repositories will be checked into the remote git server(s). 

After you use the tool several times, you will feel comfortable to run the task only once with checkin enabled in the first run. 
