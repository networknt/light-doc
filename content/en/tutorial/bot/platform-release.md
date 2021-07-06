---
title: "Light Release"
date: 2018-10-05T09:02:16-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light consists of numeric repositories in github.com/networknt organization. Some of the repositories are libraries and some of them are applications built on top of other libraries. For libraries, they need to be released to maven central, and for applications, they need to be released to docker hub or both. Before we have [light-bot][], the manual release takes several days for testing and repeating the cycle. Sometimes we have to abort a release number as a defect is identified at a late stage of the release cycle. 

The problem we were facing is common for large-scale microservices DevOps. There are too many repositories with libraries and applications with complex dependencies. Existing DevOps tools are focusing on a single repository and none of them can handler the integration test and multiple repositories or even organizations release process. 

To make ours and our users lives easier, we have developed light-bot which is a DevOps pipeline that is focusing on microservices. Now, we are running the integration test continuously to ensure the quality of the codebase. And at release time, we just need to issue several commands on the terminal. The release notes are generated and uploaded to github.com repositories, but sometimes, we need to update it to add an upgrade guideline section for certain repositories. This is the only manual work that we cannot automate. 

The following is a tutorial that walks through the release cycle. It allows our users to peek into it and learn how to leverage the same tool for their microservices projects. 

### Prerequisites

* Gpg key must be installed on the release machine for maven release. 
* Github token environment variable CHANGELOG_GITHUB_TOKEN to allow release note generation.
* Clone and build light-bot by following this [build light-bot] tutorial.
* Clone [light-config-test][] repository which contains the light-bot config files to the same workspace. 
* Start docker-compose-integration-test.yml and docker-compose-cdcserver-for-tram.yml from light-docker

### Maven Release

All libraries must be release to maven central. The following is the steps to trigger the release. 

#### Release Number
Update the light-config-test/light-bot/release-maven/release-all/config/release-maven.yml for new release number. 

Here is the entire config file. 

```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasemaven
# release version that is used to generate changelog. This need to be changed every time
version: 1.5.20
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# You can skip checkout if you are sure that the code in workspace are the latest and
# you just want to repeat the release process due to some environmental issue before.
skip_checkout: false
# You can skip the merge step is this has been done in the previous execution.
skip_merge: false
# You can skip the last release step so that you can double check the merged result.
skip_release: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  - branch: develop
    repository:
    - git@github.com:networknt/light-4j.git
    - git@github.com:networknt/json-overlay.git
    - git@github.com:networknt/openapi-parser.git
    - git@github.com:networknt/light-rest-4j.git
    - git@github.com:networknt/light-graphql-4j.git
    - git@github.com:networknt/light-hybrid-4j.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-saga-4j.git
    - git@github.com:networknt/light-session-4j.git
    - git@github.com:networknt/light-spa-4j.git
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-tokenization.git
    - git@github.com:networknt/light-consumer-4j.git
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
# merge develop to master and check in master for above repositories.
# It assumes that you always merge into master branch to release.
# Some of the repositories above are not going to be release but simple merge from
# develop to master in order to sync with other repo's release.
merge:
# generate changelog.md from github issues and check in
# release to maven central
# merge the changelog.md to develop branch
# publish the release to github repository
release:
  - light-4j
  - json-overlay
  - openapi-parser
  - light-rest-4j
  - light-graphql-4j
  - light-hybrid-4j
  - light-codegen
  - light-eventuate-4j
  - light-tram-4j
  - light-saga-4j
  - light-session-4j
  - light-spa-4j
  - light-consumer-4j
#  - light-proxy
#  - light-router
#  - light-oauth2

```

As you can see that the release number has been update to 1.5.20 release. 

#### Release Command

From your workspace

```
cd light-config-test/light-bot/release-maven/release-all
./run.sh
```

Depending on your computer and network speed, it might take 30 to 40 minutes to complete. 

After the release is done, you might need to update the generated release notes to add a section for upgrade guideline. 

### Docker Release

There are a lot of infrastructure services provided as part of the light platform to support large scale microservices deployment. There services are released as docker images in docker hub. In the entire release cycle, the docker release should be done after the maven release as these applications/services are depending on the cross-cutting concerns released to maven. 

#### Release Number

First we need to update the release vesion in the config file located at

```
light-config-test/light-bot/release-docker/release-all/config
```

Here is the update config with release 1.5.20

```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasedocker
# new upgraded version. This needs to be changed for every release.
version: 1.5.20
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# skip checkout if you know all latest code has been checkout in the previous build
skip_checkout: false
# skip merge if develop branch has been merged to master
skip_merge: false
# skip maven build if you know the build has been done already
skip_maven: false
# skip release if it is already done.
skip_release: false
# skip docker publish if it is done already.
skip_docker: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  # here we checkout master branch to publish the images. 
  - branch: develop
    repository:
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-tokenization.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-example-4j.git
    # - git@github.com:networknt/light-codegen.git
    # - git@github.com:networknt/light-docker.git
# merge develop to master for above repos
merge:
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-portal
  - light-eventuate-4j
  - light-tram-4j
  - light-example-4j
  # - light-codegen
  # - light-docker
# maven build
maven:
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-portal
  - light-eventuate-4j
  - light-tram-4j
  - light-example-4j/discovery/api_a/kubernetes
  - light-example-4j/discovery/api_b/kubernetes
  - light-example-4j/discovery/api_c/kubernetes
  - light-example-4j/discovery/api_d/kubernetes
  # - light-codegen

# update changelog in master, merge it to develop and publish to github.com
# please note that light-eventuate-4j and light-tram-4j are released in release-maven task.
release:
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-portal
  # - light-codegen

# build docker image and push to docker hub
docker:
  - light-proxy
  - light-router
  - light-oauth2/authorize
  - light-oauth2/client
  - light-oauth2/code
  - light-oauth2/key
  - light-oauth2/refresh-token
  - light-oauth2/service
  - light-oauth2/token
  - light-oauth2/user
  - light-tokenization
  - light-portal/hybrid-command
  - light-portal/hybrid-query
  - light-example-4j/discovery/api_a/kubernetes
  - light-example-4j/discovery/api_b/kubernetes
  - light-example-4j/discovery/api_c/kubernetes
  - light-example-4j/discovery/api_d/kubernetes
  # need to shutdown docker-compose-integration-test.yml before building the following two images.
  # moved to release-cdc sub task.
  # - light-eventuate-4j/eventuate-cdc-server
  # - light-tram-4j/tram-cdc-mysql-server
  # - light-docker/light-codegen

```

#### Command Line

Now let's run the command line to start light-bot. 

```
cd light-config-test/light-bot/release-docker/release-all
./run.sh
```

It will take about 30 minutes to get all docker images released. 

### Release light-codegen Docker

Light-codegen needs to be built within a docker container so it has to wait all maven libraries are available on maven central before starting it. Normally it takes about 30 minutes to several hours to have all the libraries ready. 

There is a separate config of light-bot for this build and it depends on light-docker to be in the same workspace. 

#### Release Number

First let's update the config release-docker.yml for the new release number. The file is located at

light-config-test/light-bot/release-docker/release-codegen/config

Here is the content after the modification


```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasedocker
# new upgraded version. This needs to be changed for every release.
version: 1.5.20
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# skip checkout if you know all latest code has been checkout in the previous build
skip_checkout: false
# skip merge if develop branch has been merged to master
skip_merge: false
# skip maven build if you know the build has been done already
skip_maven: false
# skip release if it is already done.
skip_release: true
# skip docker publish if it is done already.
skip_docker: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  # here we checkout master branch to publish the images. 
  - branch: develop
    repository:
    # - git@github.com:networknt/light-proxy.git
    # - git@github.com:networknt/light-router.git
    # - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-docker.git
# merge develop to master for above repos
merge:
  # - light-proxy
  # - light-router
  # - light-oauth2
  - light-codegen
  - light-docker
# maven build
maven:
  # - light-proxy
  # - light-router
  # - light-oauth2
  - light-codegen

# update changelog in master, merge it to develop and publish to github.com
release:
  # - light-proxy
  # - light-router
  # - light-oauth2
  # - light-codegen

# build docker image and push to docker hub
docker:
  # - light-proxy
  # - light-router
  - light-docker/light-codegen
  # - light-oauth2/authorize
  # - light-oauth2/client
  # - light-oauth2/code
  # - light-oauth2/key
  # - light-oauth2/refresh-token
  # - light-oauth2/service
  # - light-oauth2/token
  # - light-oauth2/user

```

As you can see the version has been changed to 1.5.20

#### Command Line

Now go to the parent folder and execute run.sh

```
cd light-config-test/light-bot/release-docker/release-codegen
./run.sh
```

As this is only one image, it will take several minutes to release it. 

### Release CDC Images

During the entire release cycle we have some infrastructure services up and running to support the local build. Two of them are light-eventuate-4j and light-tram-4j cdc servers. There servers are deployed as docker images but we have to shutdown the docker-compose before create the new docker images. 

Here is the docker containers running during thet release. 

```
 docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
3630a5df235e        networknt/tram-cdcserver:latest        "/bin/sh -c 'java -D…"   16 hours ago        Up 16 hours                                                                    lightdocker_tram-cdcserver_1
7ed60015bd59        networknt/eventuate-cdcserver:latest   "/bin/sh -c 'java -D…"   16 hours ago        Up 16 hours                                                                    lightdocker_eventuate-cdcserver_1
02095c08c2e2        networknt/kafka:latest                 "/bin/sh -c ./run-ka…"   16 hours ago        Up 16 hours         0.0.0.0:9092->9092/tcp                                     lightdocker_kafka_1
b4b2fd840a27        networknt/zookeeper:latest             "/docker-entrypoint.…"   16 hours ago        Up 16 hours         0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 3888/tcp   lightdocker_zookeeper_1
8f21e0d4a152        redis                                  "docker-entrypoint.s…"   16 hours ago        Up 16 hours         0.0.0.0:6379->6379/tcp                                     lightdocker_redis_1
3d6b7f6201bd        networknt/mysql                        "docker-entrypoint.s…"   16 hours ago        Up 16 hours         0.0.0.0:3306->3306/tcp, 33060/tcp                          lightdocker_mysql_1
daf8d896f0a8        arangodb/arangodb:latest               "/entrypoint.sh aran…"   16 hours ago        Up 16 hours         0.0.0.0:8529->8529/tcp                                     lightdocker_arango_1

```

The following commands are used to shutdown down from light-docker folder. 

```
docker-compose -f docker-compose-cdcserver-for-tram.yml down
docker-compose -f docker-compose-integration-test.yml down
```

User docker ps to check if all contains are down. 

#### Release Number

Like other docker releases, we need to update the release number first in the release-docker.yml config file. It is located at the following folder. 

light-config-test/light-bot/release-docker/release-cdc/config

Here is the updated file with release number 1.5.20

```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasedocker
# new upgraded version. This needs to be changed for every release.
version: 1.5.20
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# skip checkout if you know all latest code has been checkout in the previous build
skip_checkout: true
# skip merge if develop branch has been merged to master
skip_merge: true
# skip maven build if you know the build has been done already
skip_maven: true
# skip release if it is already done.
skip_release: true
# skip docker publish if it is done already.
skip_docker: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  # here we checkout master branch to publish the images. 
  - branch: develop
    repository:
    # - git@github.com:networknt/light-proxy.git
    # - git@github.com:networknt/light-router.git
    # - git@github.com:networknt/light-oauth2.git
    # - git@github.com:networknt/light-tokenization.git
    # - git@github.com:networknt/light-portal.git
    # - git@github.com:networknt/light-eventuate-4j.git
    # - git@github.com:networknt/light-tram-4j.git
    # - git@github.com:networknt/light-example-4j.git
    # - git@198.55.49.186:lightapi/tokenization.git
    # - git@github.com:networknt/light-codegen.git
    # - git@github.com:networknt/light-docker.git
# merge develop to master for above repos
merge:
  # - light-proxy
  # - light-router
  # - light-oauth2
  # - light-tokenization
  # - light-portal
  - light-eventuate-4j
  - light-tram-4j
  # - tokenization
  # - light-example-4j
  # - light-codegen
  # - light-docker
# maven build
maven:
  # - light-proxy
  # - light-router
  # - light-oauth2
  # - light-tokenization
  # - light-portal
  # - light-eventuate-4j
  # - light-tram-4j
  # - light-example-4j/discovery/api_a/kubernetes
  # - light-example-4j/discovery/api_b/kubernetes
  # - light-example-4j/discovery/api_c/kubernetes
  # - light-example-4j/discovery/api_d/kubernetes
  # - tokenization
  # - light-codegen

# update changelog in master, merge it to develop and publish to github.com
# please note that light-eventuate-4j and light-tram-4j are released in release-maven task.
release:
  # - light-proxy
  # - light-router
  # - light-oauth2
  # - light-tokenization
  # - light-portal
  # - tokenization
  # - light-codegen

# build docker image and push to docker hub
docker:
  # - light-proxy
  # - light-router
  # - light-oauth2/authorize
  # - light-oauth2/client
  # - light-oauth2/code
  # - light-oauth2/key
  # - light-oauth2/refresh-token
  # - light-oauth2/service
  # - light-oauth2/token
  # - light-oauth2/user
  # - light-tokenization
  # - light-portal/hybrid-command
  # - light-portal/hybrid-query
  # - light-example-4j/discovery/api_a/kubernetes
  # - light-example-4j/discovery/api_b/kubernetes
  # - light-example-4j/discovery/api_c/kubernetes
  # - light-example-4j/discovery/api_d/kubernetes
  # need to shutdown docker-compose-integration-test.yml before building the following two images.
  - light-eventuate-4j/eventuate-cdc-server
  - light-tram-4j/tram-cdc-mysql-server
  # - tokenization
  # - light-docker/light-codegen

```

#### Command Line

```
cd light-config-test/light-bot/release-docker/release-cdc
./run.sh
```

It will take a few minutes to get two images released. 

### Upgrade Version

Once everything is released, we need to update the develop branch for each repositories to the next development version. To do that go to the following location.

light-config-test/light-bot/version-upgrade/upgrade-all

Here is the updated config file with version upgraded from 1.2.20 to 1.2.21. You need to change three places for the github comment, old version number and new version number. Here is the updated config file. 

```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: versionupgrade
# git commit comment for this task
comment: 'upgrade to version 1.5.21'
# old existing version that need to be upgraded
old_version: 1.5.20
# new upgraded version. These two versions need to be updated every time to run this command
new_version: 1.5.21
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
  - branch: develop
    repository:
    - git@github.com:networknt/light-4j.git
    - git@github.com:networknt/json-overlay.git
    - git@github.com:networknt/openapi-parser.git
    - git@github.com:networknt/light-rest-4j.git
    - git@github.com:networknt/light-graphql-4j.git
    - git@github.com:networknt/light-hybrid-4j.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-eventuate-4j.git
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
  - path: light-graphql-4j/src/main/resources/templates/graphql/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-hybrid-4j/src/main/resources/templates/hybrid/server/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-hybrid-4j/src/main/resources/templates/hybrid/service/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.json-overlay>\d*\.\d*\.\d*</version.json-overlay>
  - path: light-rest-4j/src/main/resources/templates/rest/openapi/pom.xml.rocker.raw
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: light-rest-4j/src/main/resources/templates/rest/swagger/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-eventuate-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-tram-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
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

The file is very long as we have numeric projects in light-example-4j that need to be upgraded to the new release number. 

Once the file is updated, we can run the command line.

```
cd light-config-test/light-bot/version-upgrade/upgrade-all
./run.sh
```

It takes several minutes to upgrade all projects in networknt and push the develop branches to github. 

### Checkout

In the previous step, we have upgraded develop branches for all repositories in github. If you are a platform develop, you might need to consider to checkout all the develop branches for each repo to your workspace. I am using networknt under my home directory as my workspace and there is a light-bot config available to checkout every projects in one short. It will pull the latest develop branch if the repository is in the workspace already. Or clone it if it doesn't exist. 

To checkout all repositories into networknt workspace. 

```
cd light-config-test/light-bot/checkout/checkout-all
./run.sh
```

There is no need to update the config file for this task as there is no version in the config. 

If there are multiple developers are working on these projects, you should run the checkout task at least very morning or even several times per day to ensure that you are always working on the latest codebase. 


### Build and Test

Now we have the new develop version checked out into the workspace, we can build and run the integration test on this new version and this process can be repeated several times per day if necessary. It should be done on a build server but it is OK to run it on you local especially after new release. 

Before running the build and test locally, we need to start all the infrastructure services that are supporting the build and integration test. You need to ensure that you have light-docker repository checked out into your workspace. 

```
cd light-docker
docker-compose -f docker-compose-integration-test.yml up -d
```

After several minutes, start the docker-compose-cdcserver-for-tram.yml file. It should be in the integration-test YAML but there is a bug in tram cdcserver that need Kafka and Zookeeper to be up and running before starting it. 

```
docker-compose -f docker-compose-cdcserver-for-tram.yml up -d
```

Now go to the following folder and run the command. 

```
cd light-config-test/light-bot/develop-build/build-test-all
./run.sh
```


This will take a while to get everything built and tested. You can repeat this process daily or even more time automatically. 


### Summary

As you can see the entire process is just updating the config file and run several commands. All the config files mentioned in this tutorial are for networknt organization release. If you have microservices scattered into multiple repositories or even multiple organizations, you just need to update the config files and can follow the same process for release. 

As light-bot is built on top of light-4j framework, all tasks are built as plugins that is wired in service.yml config file. That means it is very easy to be extended to meet your specific requirement. 

Along with the release pipeline, there are other tools/pipelines to support shifting DevOps to the left. The integation for micorservices should be started the first endpoint is developed, not when you complete the entire service.



[light-bot]: /tool/light-bot/
[build light-bot]: /tutorial/bot/build-light-bot/
[light-config-test]: https://github.com/networknt/light-config-test
