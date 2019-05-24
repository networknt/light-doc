---
title: "Version Upgrade"
date: 2019-03-27T13:49:47-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This should be called right after a new release so that all repository versions can be upgraded to the next one in one shot and check in into each release branch. 

Here are the steps in the process. 

* clone all projects to a workspace defined in the version-upgrade.yml in your home directory and switch to the release branch specified.
* if the workspace exists, then switch each project to the release branch and pull from remote to make sure you have the latest.
* run "mvn versions:set -DnewVersion=1.5.6 -DgenerateBackupPoms=false" to update pom.xml versions in each project
* update other files that contain the version with regex defined in the config file.
* check in and push the release branch

For example configuration files, please visit [light-config-test][]. 

Here is an example of version-upgrade.yml used for networknt repositories in master branch.

```yaml
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: versionupgrade_1_6_x
# git commit comment for this task
comment: 'upgrade to version 1.6.0'
# old existing version that need to be upgraded
old_version: 1.5.32
# new upgraded version. These two versions need to be updated every time to run this command
new_version: 1.6.0
# only skip the checkout if you are sure that the latest code is in the workspace. It is
# OK to execute it several times.
skip_checkout: false
# skip maven version upgrade. It should not be skip if you haven't run it yet. It is OK
# to execute it several times.
skip_maven: false
# skip the regular expression match and replace. It is safe to run it multiple times.
skip_version: false
# skip checkin if you want to review the changes before checking in to github.
skip_checkin: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  - branch: master
    repository:
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
    - git@github.com:networknt/light-consumer-4j.git
    - git@github.com:networknt/light-config-server.git
    - git@github.com:networknt/http2client-benchmark.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/light-spring-boot.git

# run mvn versions:set command in the following folders.
maven:
  - light-4j
  - json-overlay
  - openapi-parser
  - light-rest-4j
  - light-graphql-4j
  - light-hybrid-4j
  - light-codegen
  - light-eventuate-4j
  - light-tram-kafka
  - light-tram-4j
  - light-saga-4j
  - light-session-4j
  - light-spa-4j
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-consumer-4j
  - light-config-server
  - light-spring-boot
  - light-portal/hybrid-command
  - light-portal/hybrid-query
# regex replacement for dependencies
version:
  json-overlay:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  openapi-parser:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  light-rest-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-graphql-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-hybrid-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-codegen:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # light-graphql-4j  
  - path: light-graphql-4j/src/main/resources/templates/graphql/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # light-hybrid-4j
  - path: light-hybrid-4j/src/main/resources/templates/hybrid/server/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-hybrid-4j/src/main/resources/templates/hybrid/service/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # light-rest-4j openapi  
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # light-rest-4j swagger
  - path: light-rest-4j/src/main/resources/templates/rest/swagger/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # light-rest-4j kotlin
  - path: light-rest-4j/src/main/resources/templates/restkotlin/gradleProperties.rocker.raw
    match: light4jVersion=\d*\.\d*\.\d*
  # light-eventuate-4j openapi
  - path: light-eventuate-4j/src/main/resources/templates/eventuate/rest/openapi/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-eventuate-4j/src/main/resources/templates/eventuate/rest/openapi/pom.xml.rocker.raw
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: light-eventuate-4j/src/main/resources/templates/eventuate/rest/openapi/pom.xml.rocker.raw
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # light-eventuate-4j hybrid
  - path: light-eventuate-4j/src/main/resources/templates/eventuate/hybrid/server/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-eventuate-4j/src/main/resources/templates/eventuate/hybrid/service/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  light-eventuate-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-tram-kafka:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-tram-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.light-tram-kafka>\d*\.\d*\.\d*</version.light-tram-kafka>
  light-saga-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-session-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-spa-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-proxy:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-router:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-oauth2:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-tokenization:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-consumer-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-config-server:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-spring-boot:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-portal:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: api-certification/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: host-menu/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: host-menu/hybrid-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: host-menu/hybrid-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: oauth-playground/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: schema-form/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: schema-form/hybrid-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: schema-form/hybrid-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/common/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/hybrid-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: user-management/hybrid-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: hybrid-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: hybrid-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  light-example-4j:
  # eventuate framework examples
  # eventuate account management
  - path: eventuate/account-management/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # eventuate todolist
  - path: eventuate/todo-list/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: eventuate/todo-list/rest-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: eventuate/todo-list/hybrid-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: eventuate/todo-list/rest-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: eventuate/todo-list/hybrid-query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # discovery
  # discovery api_a
  - path: discovery/api_a/consul-tls/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/http-health/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/consul/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/consuldocker/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/dynamic/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/multiple/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/static/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/tag/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/token/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_a/kubernetes/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # discovery api_b
  - path: discovery/api_b/consul-tls/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/http-health/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/consul/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/consuldocker/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/dynamic/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/multiple/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/static/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/tag/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/token/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_b/kubernetes/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # discovery api_c
  - path: discovery/api_c/consul-tls/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/http-health/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/consul/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/consuldocker/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/dynamic/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/multiple/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/static/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/tag/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/token/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_c/kubernetes/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # discovery api_d
  - path: discovery/api_d/consul-tls/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/http-health/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/consul/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/consuldocker/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/dynamic/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/multiple/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/static/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/tag/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/token/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: discovery/api_d/kubernetes/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # multidb
  - path: common/multidb/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: common/multidb/generated/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: common/multidb/generated/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: common/multidb/dbconfig/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: common/multidb/dbconfig/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: common/multidb/dbconfig/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # router-demo
  - path: spa/react-stateless/router-demo/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: spa/react-stateless/router-demo/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # secspa
  - path: graphql/secspa/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>


  # graphql framework
  # graphql subscription
  - path: graphql/subscription/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql helloworld
  - path: graphql/helloworld/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql mutation-idl
  - path: graphql/mutation-idl/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql relaytodo
  - path: graphql/relaytodo/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql mutation
  - path: graphql/mutation/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql starwars
  - path: graphql/starwars/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # graphql secspa
  - path: graphql/secspa/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # light-tram-4j framework
  # tram todolist
  - path: tram/light-tram-todolist/multi-module/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: tram/light-tram-todolist/multi-module/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: tram/light-tram-todolist/multi-module/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # tram todolist command
  - path: tram/light-tram-todolist/multi-module/tram-todo-command/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: tram/light-tram-todolist/multi-module/tram-todo-command/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: tram/light-tram-todolist/multi-module/tram-todo-command/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # tram todolist view
  - path: tram/light-tram-todolist/multi-module/tram-todo-view/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: tram/light-tram-todolist/multi-module/tram-todo-view/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: tram/light-tram-todolist/multi-module/tram-todo-view/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # tram todolist single module
  - path: tram/light-tram-todolist/single-module/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: tram/light-tram-todolist/single-module/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: tram/light-tram-todolist/single-module/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  # light-saga-4j framework
  # saga customers orders
  - path: saga/light-saga-cutomers-and-orders/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: saga/light-saga-cutomers-and-orders/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: saga/light-saga-cutomers-and-orders/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # saga customers orders order service
  - path: saga/light-saga-cutomers-and-orders/order-service/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: saga/light-saga-cutomers-and-orders/order-service/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: saga/light-saga-cutomers-and-orders/order-service/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  # saga customers orders customer service
  - path: saga/light-saga-cutomers-and-orders/customer-service/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: saga/light-saga-cutomers-and-orders/customer-service/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: saga/light-saga-cutomers-and-orders/customer-service/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  # light-rest-4j framework
  # swagger
  # proxy-backend
  - path: rest/swagger/proxy-backend/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # petstore
  - path: rest/swagger/petstore/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # ms-chain
  - path: rest/swagger/ms_chain/api_a/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_a/metrics/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_a/httpschain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_a/security/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_a/httpchain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: rest/swagger/ms_chain/api_b/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_b/metrics/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_b/httpschain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_b/security/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_b/httpchain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: rest/swagger/ms_chain/api_c/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_c/metrics/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_c/httpschain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_c/security/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_c/httpchain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: rest/swagger/ms_chain/api_d/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_d/metrics/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_d/httpschain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_d/security/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/ms_chain/api_d/httpchain/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  # database
  - path: rest/swagger/database/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/connection/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/oracle/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/postgres/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/queries/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/query/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/test/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/swagger/database/updates/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # openapi
  - path: rest/openapi/proxy-backend/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/proxy-backend/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/proxy-backend/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: rest/openapi/cors/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/cors/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/cors/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: rest/openapi/petstore/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/petstore/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/petstore/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: rest/openapi/ms-aggregate/aa/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/aa/generated/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/aa/generated/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ab/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ab/generated/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ab/generated/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ac/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ac/generated/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ac/generated/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ad/generated/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ad/generated/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ad/generated/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: rest/openapi/ms-aggregate/aa/https/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/aa/https/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/aa/https/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ab/https/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ab/https/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ab/https/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ac/https/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ac/https/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ac/https/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: rest/openapi/ms-aggregate/ad/https/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: rest/openapi/ms-aggregate/ad/https/pom.xml
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: rest/openapi/ms-aggregate/ad/https/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>


  # client
  - path: client/standalone/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: client/tomcat/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: client/tableau/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  - path: client/consul/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # webserver
  - path: webserver/api-simple-web/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # middleware-performance
  - path: middleware-performance/service-config/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: middleware-performance/service-config/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: middleware-performance/endpoint-individual/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: middleware-performance/endpoint-individual/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>

  - path: middleware-performance/endpoint-source/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: middleware-performance/endpoint-source/pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  

  # http2client-benchmark
  http2client-benchmark:
  - path: httpserver/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: lightclient/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

  # microservices-framework-benchmark
  microservices-framework-benchmark:
  - path: light-4j/pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>

```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/version-upgrade
