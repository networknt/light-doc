---
title: "Light Bot"
date: 2017-12-30T07:15:31-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tool"
    weight: 80
weight: 80	#rem
aliases: []
toc: false
draft: false
---

One of the design principles of light*4j frameworks is to make each Lego piece smaller so
that developers have the flexibility to compose their services with just enough functionality
and smaller memory footprint/smaller delivery package. As you can see, most light-4j cross-cutting 
modules only have about five Java classes on average and we have a lot of modules to manage.

It is even more complicated since we support different [style of interaction][] for microservices
and we have to create several frameworks as different customers are using different frameworks.

This has huge negative impact on package management, testing and release process as there are
a lot more moving pieces. Every time we have a release of frameworks, there are at least two days
in testing in order to make sure the everything works together. 

We have been looking at Devops tools on the open source world in order to streamline our process
but all of the tools are not designed for us. They are focus on single repository monolithic app
only as most of them build single project by putting a config file in the repository. 

We need a automation tool that can handler multiple repos with dependencies for our own packages
and at the same time, our customers are asking for a similar tool to manage their microservices
once the numbers are increasing. 

This forces us to consider building our own Devops tool [light-bot][]  

To start a full-blown devops tool chain is not an easy job and we are starting from command line
first and will add central control server and agent server later on. 

Another goal for this light-bot project is to try Gradle with [Kotlin DSL][]. As you know,
light-4j frameworks are built with Maven and I have been looking at Gradle for a while but
don't want to learn Groove DSL. With release 4.4.1, the Kotlin DSL is good enough for production
usage and I am trying it with light-bot project. So far everything works perfectly and it is
really fast compare with Maven build. The next step is to update light-codegen to give user
option to choose Maven or Gradle in the config.json file.


To build the project, just run "./gradlew build" in the root folder of light-bot.

For more information on how to use light-bot in your project, please refer to [light-bot tutorial][]

### exec-core

This is the core execution module that contains small command line executors and a wrapper of
Java process API. 

### bot-cli

A command line utility which is an entry point to call all the major tasks below.

To run the command line, go to "~/networknt/light-bot/bot-cli/build/libs" folder

* java -jar bot-cli-fat-1.0.jar -t develop 
* java -jar bot-cli-fat-1.0.jar -t version
* java -jar bot-cli-fat-1.0.jar -t release 

### exec-develop

This is a develop branch build and test process. It should be setup as a scheduled process on
build server for now and in the future it should be called by agent server once controller
receives push event from github.com 

Before running the following build for networknt repositories, you need to start the following
dependencies as integration tests need these dependencies. 

* start docker-compose-integration-test.yml from light-docker
* start docker-compose-cdcserver-for-tram.yml from light-docker

Here are the steps in the process.

* it clones all projects define in the checkout section into a workspace.
* if the workspace exists, then it will iterate each repository to switch to the develop branch and pull from remote. If none of the repo has been updated, then skip the rest of process.
* build each repository with "maven clean install". This command runs unit tests and integration tests for each repo.
* start one or more microservices and run end-to-end test cases.


Here is an example of develop.yml used for networknt repositories and examples.

```yaml
workspace: workspace
checkout:
  branch: develop
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
  - git@github.com:networknt/light-oauth2.git
  - git@github.com:networknt/light-example-4j.git
  - git@github.com:networknt/light-docker.git
build:
- light-4j
- openapi-parser
- light-rest-4j
- light-graphql-4j
- light-hybrid-4j
- light-codegen
- light-eventuate-4j
- light-tram-4j
- light-saga-4j
- light-session-4j
- light-proxy
- light-oauth2
- light-example-4j/rest/swagger/petstore
- light-example-4j/rest/openapi/petstore
- light-example-4j/rest/swagger/proxy-backend
- light-example-4j/rest/openapi/proxy-backend
- light-example-4j/rest/openapi/cors
- light-example-4j/rest/swagger/ms_chain/api_a/httpschain
- light-example-4j/rest/swagger/ms_chain/api_b/httpschain
- light-example-4j/rest/swagger/ms_chain/api_c/httpschain
- light-example-4j/rest/swagger/ms_chain/api_d/httpschain
test:
  rest-swagger-petstore:
    server:
    - path: light-example-4j/rest/swagger/petstore
      cmd: target/petstore-1.0.1.jar
    request:
    - host: https://localhost:8443
      path: "/v2/pet/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": doggie
          "$.id": 123456789
  rest-openapi-petstore:
    server:
    - path: light-example-4j/rest/openapi/petstore
      cmd: target/petstore-1.0.1.jar
    request:
    - host: https://localhost:8443
      path: "/v1/pets/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": Jessica Right
          "$.id": 1
  rest-swagger-proxy-backend:
    server:
    - path: light-example-4j/rest/swagger/proxy-backend
      cmd: target/backend-1.0.0.jar
    request:
    - host: https://localhost:8081
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8081
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
  rest-openapi-proxy-backend:
    server:
    - path: light-example-4j/rest/openapi/proxy-backend
      cmd: target/backend-1.0.0.jar
    request:
    - host: https://localhost:8081
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8081
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
  rest-openapi-cors:
    server:
    - path: light-example-4j/rest/openapi/cors
      cmd: target/cors-1.0.1.jar
    request:
    - host: https://localhost:8443
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.[0].key": key1
          "$.[1].key": key2
          "$.[0].value": value1
          "$.[1].value": value2
    - host: https://localhost:8443
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8443
      path: "/v1/postData"
      method: options
      header:
        "Origin": http://example.com
        "Access-Control-Request-Method": POST
        "Access-Control-Request-Headers": X-Requested-With
      response:
        status: 200
  rest-swagger-chain:
    server:
    - path: light-example-4j/rest/swagger/ms_chain/api_d/httpschain
      cmd: target/apid-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_c/httpschain
      cmd: target/apic-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_b/httpschain
      cmd: target/apib-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_a/httpschain
      cmd: target/apia-1.0.0.jar
    request:
    - host: https://localhost:7441
      path: "/v1/data"
      method: get
      response:
        status: 200
        body:
          "$.length()": 8

```

### exec-version

This should be called right after a new release so that all repository versions can be upgrade
to the next one in one shot and check in into develop branch. The process is controlled by a
version.yml file.

Here are the steps in the process. 

* clone all projects to a workspace (default as version) in your home directory and switch to develop branch.
* if the workspace exists, then switch each project to develop branch and pull from remote to make sure you have the latest.
* run "mvn versions:set -DnewVersion=1.5.6 -DgenerateBackupPoms=false" to update pom.xml versions in each project
* update other files that contains the version with regex defined in the config file.
* check in and push develop branch

Here is an example of version.yml used for networknt repositories.

```yaml
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: version
# old existing version that need to be upgraded
old_version: 1.5.5
# new upgraded version. These two versions need to be updated every time to run this command
new_version: 1.5.6
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  branch: develop
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
  - git@github.com:networknt/light-oauth2.git
  - git@github.com:networknt/light-example-4j.git
  - git@github.com:networknt/light-docker.git
# run mvn versions:set command in the following folders.
maven:
  - light-4j
  - openapi-parser
  - light-rest-4j
  - light-graphql-4j
  - light-hybrid-4j
  - light-codegen
  - light-eventuate-4j
  - light-tram-4j
  - light-saga-4j
  - light-session-4j
  - light-proxy
  - light-oauth2
# regex replacement for dependencies
version:
  openapi-parser:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-rest-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
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
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  - path: light-rest-4j/src/main/resources/templates/rest/swagger/pom.xml.rocker.raw
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  light-eventuate-4j:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: docs/content/tutorial/todo-list.md
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
  light-proxy:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
  - path: pom.xml
    match: <version.openapi-parser>\d*\.\d*\.\d*</version.openapi-parser>
  light-oauth2:
  - path: pom.xml
    match: <version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>


```

### exec-release

This is the task to release all frameworks and modules all together. It first checkout repos
into a workspace and switch to develop defined in release.yml file. If the workspace exists,
then it will switch to the develop and run git pull origin develop to ensure the codebase is
the same as remote.

Then the release process is started with the following step for each repository.

* merge develop branch to master and check it in.
* generate CHANGELOG.md from github.com and check into master.
* run "mvn clean install deploy -DperformRelease" to release to Maven Central.
* merge the CHANGELOG.md to develop branch and push to remote.
* read CHANGELOG.md for the current release into a StringBuffer.
* call github API to create a new release and tag 

Note that before release, all projects should be build successfully in develop branch by
exec-develop. After the release, you might need to change the github release a little bit
to add upgrade guide for example. 

Here is an example of release.yml config file for networknt projects.

```yaml
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: release
# release version that is used to generate changelog. This need to be changed every time
version: 1.5.6
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  branch: develop
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
  - git@github.com:networknt/light-oauth2.git
  - git@github.com:networknt/light-example-4j.git
  - git@github.com:networknt/light-docker.git
# merge develop to master, check in master and release master
release:
  - light-4j
  - openapi-parser
  - light-rest-4j
  - light-graphql-4j
  - light-hybrid-4j
  - light-codegen
  - light-eventuate-4j
  - light-tram-4j
  - light-saga-4j
  - light-session-4j
#  - light-proxy
#  - light-oauth2

```

### external dependencies

If you look at the code, you will realized that the light-bot is an command line executor which
leverage the Linux/Mac bash to its extreme. We are just using Java code to glue all most useful
command lines together to automate them in a way we think the most efficient. In order to do that
we are depending on certain setups on the local environment or your devops servers.

##### JDK, Maven

As most projects in networknt are built in Java and compiled by Maven, we need to ensure that
JDK 1.8 and Maven 3.5.x are installed. Use the following commands to check if they are installed
correctly.


```
java -version
mvn --version
```

##### Git

Unlike other devops tools that support multiple VCS systems, we only support git and for some of
the features like create release note, only github.com or compatible gogs are supported. As we
are calling git command line most of the time, we need to ensure that ssh key pair are created
on your host and upload the public key to github.com or gogs server. 

##### Docker

Docker needs to be available to start some of the depending services for building and testing. As
you can see in the above steps, we are using docker-compose to start Kafka, Zookeeper, Mysql, Redis
and Cdc server for light-tram-4j

Also, we are using [networknt/github-changelog-generator][] to generate changelog from github pull
request and issues. 

##### Email

For develop branch build and test, there is an email notification can be sent out if anything is
wrong. In order to use it, you have to setup the email.yml to connect to your email server with
correct credentials. The default config is using noreply@lightapi.net and you have to change to 
an account available on your email server.



[style of interaction]: /style/
[light-bot]: https://github.com/networknt/light-bot
[Kotlin DSL]: https://github.com/gradle/kotlin-dsl
[light-bot tutorial]: /tutorial/bot/
[networknt/github-changelog-generator]: https://github.com/skywinder/github-changelog-generator
