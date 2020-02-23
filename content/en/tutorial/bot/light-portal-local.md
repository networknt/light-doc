---
title: "Light Portal Local"
date: 2018-03-31T07:19:14-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-portal contains too many independent microservices, and it is very tedious to build them one by one. To make the developer's life easier, we have created a light-bot develop-build config in the [light-config-test][] repository.

Before using light-bot, you need to build it locally by following [build light-bot][] tutorial. 

Once you have light-bot built inside `~/networknt` workspace, the next step is to build light-portal with the command line tool provided by light-bot. 

First, let's check out the light-config-test repository, which contains various configuration folders for different builds used by light-bot.

```
cd ~/networknt
git clone https://github.com/networknt/light-config-test.git
cd light-config-test/light-bot/develop-build/build-portal
./run.sh
```


The above command line uses the following develop-build.yml configuration, and you can customize it as you wish.

```
workspace: networknt
# indicate if you want to skip checkout. yes if you know that all repositories are just checkout
# or the last time the build was failed and you just want to retry without changing anything.
skip_checkout: true
# Just checkout the repositories for backup or some other tasks
skip_build: true
# If this value is set to true, then only checkout and build will be called. It is
# very useful if you just want to install the develop branch modules into your .m2
# local repo. Also, some of our customers have constraint test environment to run
# unit and integration test only and another environment to run all of tests.
skip_test: true
# skip copy file if necessary.
skip_copyFile: false
checkout:
  branch: develop
  repository:
  - git@github.com:networknt/light-4j.git
  - git@github.com:networknt/light-hybrid-4j.git
  - git@github.com:networknt/openapi-parser.git
  - git@github.com:networknt/light-codegen.git
  - git@github.com:networknt/light-eventuate-4j.git
  - git@github.com:networknt/light-portal.git
build:
  project:
  - light-4j
  - light-hybrid-4j
  - openapi-parser
  - light-codegen
  - light-eventuate-4j
  - light-portal
  - light-portal/api-certification
  - light-portal/oauth-playground
  - light-portal/host-menu
  - light-portal/schema-form
  - light-portal/user-management
# This section defines end-to-end tests with real live servers, if you want to skip
# these tests, please change the same level skipE2ETest to true.
test:

copyFile:
  # populate hybrid-query services
  - src: light-portal/api-certification/target/api-certification-1.0.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/api-certification-1.0.0.jar
  - src: light-portal/oauth-playground/target/playground-1.0.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/playground-1.0.0.jar
  - src: light-codegen//codegen-core/target/codegen-core-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/codegen-core-1.5.11.jar
  - src: light-codegen/codegen-fwk/target/codegen-fwk-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/codegen-fwk-1.5.11.jar
  - src: light-codegen/light-graphql-4j/target/light-graphql-4j-generator-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/light-graphql-4j-generator-1.5.11.jar
  - src: light-codegen/light-rest-4j/target/light-rest-4j-generator-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/light-rest-4j-generator-1.5.11.jar
  - src: light-codegen/light-hybrid-4j/target/light-hybrid-4j-generator-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/light-hybrid-4j-generator-1.5.11.jar
  - src: light-codegen/codegen-web/target/codegen-web-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/codegen-web-1.5.11.jar
  - src: light-portal/common-util/target/common-util-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/common-util-0.1.0.jar
  - src: light-portal/host-menu/common/target/host-menu-common-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/host-menu-common-0.1.0.jar
  - src: light-portal/host-menu/command/target/host-menu-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/host-menu-command-0.1.0.jar
  - src: light-portal/hybrid-query/target/hybrid-query-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-query/service/hybrid-query-1.5.11.jar
  - src: light-portal/host-menu/query/target/host-menu-query-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/host-menu-query-0.1.0.jar
  - src: light-portal/host-menu/hybrid-query/target/menu-query-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/menu-query-0.1.0.jar
  - src: light-portal/schema-form/common/target/schema-form-common-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/schema-form-common-0.1.0.jar
  - src: light-portal/schema-form/command/target/schema-form-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/schema-form-command-0.1.0.jar
  - src: light-portal/schema-form/query/target/schema-form-query-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/schema-form-query-0.1.0.jar
  - src: light-portal/schema-form/hybrid-query/target/form-query-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-query/service/form-query-0.1.0.jar
  # populate hybrid-command services
  - src: light-portal/hybrid-command/target/hybrid-command-1.5.11.jar
    dst: light-config-test/light-portal/hybrid-command/service/hybrid-command-1.5.11.jar
  - src: light-portal/host-menu/common/target/host-menu-common-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/host-menu-common-0.1.0.jar
  - src: light-portal/host-menu/command/target/host-menu-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/host-menu-command-0.1.0.jar
  - src: light-portal/host-menu/hybrid-command/target/menu-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/menu-command-0.1.0.jar
  - src: light-portal/schema-form/common/target/schema-form-common-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/schema-form-common-0.1.0.jar
  - src: light-portal/schema-form/command/target/schema-form-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/schema-form-command-0.1.0.jar
  - src: light-portal/schema-form/hybrid-command/target/form-command-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/form-command-0.1.0.jar
  - src: light-portal/user-management/hybrid-service/target/user-hybrid-service-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/user-hybrid-service-0.1.0.jar
  - src: light-portal/user-management/hybrid-service/target/user-hybrid-service-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/user-hybrid-service-0.1.0.jar
  - src: light-portal/user-management/auth/target/usermanagement-auth-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/usermanagement-auth-0.1.0.jar
  - src: light-portal/user-management/common/target/user-management-common-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/user-management-common-0.1.0.jar
  - src: light-portal/user-management/jdbc/target/user-management-jdbc-0.1.0.jar
    dst: light-config-test/light-portal/hybrid-command/service/user-management-jdbc-0.1.0.jar


```


The above config file triggers the light-bot develop-build task to check out related repositories into the networknt workspace and switch to develop branch. If these repositories are already in the workspace, the light-bot tries to pull from the remote GitHub server to ensure that only the latest code is rebuilt. If nothing is changed after pulling from remote git server and skip_checkout is false, light-bot quits immediately. 

If you want to rebuild all repositories even there is no change, the skip_checkout flag can be set as true to skip this step. 

After the build is done, all artifacts are copied to ~/networknt/light-config-test/light-portal/hybrid-query/service or ~/networknt/light-config-test/light-portal/hybrid-command/service folder depending on if the service is command side or query side. 

Now you have light-portal built in your `~/networknt` workspace. To start the light-portal services, you can follow [start light-portal service][] tutorial. 

[build light-bot]: /tutorial/bot/build-light-bot/
[start light-portal service]: /tutorial/portal/start-portal-service/
[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/develop-build/build-portal

