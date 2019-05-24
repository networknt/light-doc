---
title: "Environment"
date: 2017-11-05T12:01:18-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 5
weight: 5
aliases: []
toc: false
draft: false
reviewed: true
---

* JDK 8 or JDK 11 (Mandatory)

Currently, the majority of the components in the light platform 1.5.x is based on Java 8, and 2.0.x is based on Java 11. Before starting your first project, we need to install the following software packages on your system. 

Most of our customers are working on light-4j production ready 1.5.x version which is depending on Java 8. We have Beta release 2.0.x on JDK 11, but some of our customers are waiting for Redhat certified JDK 11 docker image on the Openshift environment. Once the image is available, we will be moving our development to the release 2.x and switch the 1.x version in maintenance mode. 

If you are building services that will be released to production very soon, then it is recommended choosing 1.5.x which the production release. 

If you want to leverage some of the features in Java 11 and your company's policy allow you to use Docker official image to deploy your application, then you can choose 2.0.x release to build your application. Please be aware that supporting tools are still in a process to migrate to JDK 11 so the road might be a little bit bumpy. 

If you want to try out both 1.x and 2.x releases, you can install [SKDMAN][] on your computer to switch JDK with a command line. 

To confirm which version of Java is installed for your terminal session on your computer:

```
java -version
```

* Maven (Optional)

Most frameworks use Maven as the default build system from [light-codegen][]. It is not necessary to install Maven locally as we have `mvnw` and maven wrapper packaged into the generated project. If you want to install Maven locally, you can follow the instructions at [maven][] tutorial.


If you install the Maven, you can run the following command to build the generated project. 

```
mvn clean package
mvn clean install
```

Otherwise, you can use the packaged `mvnw` to build.

```
./mvnw clean package
./mvnw clean install
```

* Gradle (Optional)

By default, some of the generators scaffold project with Gradle (Kotlin DSL) build system. As `gradle wrapper` is packaged inside the generated project, you don't need to install Gradle locally. 

To build the genreated project:

```
./gradlew clean build
```

* Git (Optional)

Git is the default version control system for light-4j, and it is highly recommended for our users; however, users can choose any other version control system they like. 

* Docker (Optional)

Most of our applications are deployed as Docker containers; however, it is not necessary to install Docker locally to build Docker images. If you want to test dockerized applications locally, please install Docker on your computer. 

## Linux

## Mac

## Windows


[SKDMAN]: /tool/sdk/
[light-codegen]: /tool/light-codegen/
[maven]: /tool/maven/


