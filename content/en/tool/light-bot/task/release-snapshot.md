---
title: "Release Snapshot"
date: 2019-04-11T23:31:18-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

It has been a long time that our developers are working on the develop branch in each repository in the networknt organization. Recently, we have to change the release process to release from three release branches to meet the requirement from our users. 

* The 1.5.x branch is the stable branch for production upgrade as only defect fixes backport to this branch. 
* The master branch is our main development branch with new features added daily and release under 1.6.x tag.
* The jdk11 branch contains the code for jdk11, and only several users are using it with 2.0.x release tag.


Now, our user's requirement is satisfied. What about our contributor? As you know, we have too many repositories, and the dependencies between repositories are complicated. For most of our platform developers, we are relying on the light-bot [develop-build][] to prepare the local environment and build all dependencies. It has been working perfectly, but it takes a lot of developers' time to build everything locally. With the contributor's team getting bigger and bigger, team members are looking into the snapshot releases. 

To prepare the snapshot release, we have taken actions in the repository arrangement and light-bot update. 

### Simplify the version properties

Previously, a lot of our repositories have version references per framework even the version is the same as all frameworks are released along with light-4j. 

We have updated all repositories and light-codegen templates to use only version.light-4j in the pom.xml

Here is the old pom.xml

```
<version.light-4j>1.6.0</version.light-4j>
<version.light-rest-4j>1.6.0</version.light-rest-4j>
```

And the new pom.xml

```
<version.light-4j>1.6.0</version.light-4j>
```

This significantly simplify the light-bot [version-upgrade][] task.


### Release branches for codegen and examples

When we update the master branch for all repositories with the snapshot version before the final release, there are three repositories that we need to handle a little bit differently. 

* light-codegen
* light-example-4j
* light-example-kotlin

The above repositories are used frequently by our users with the source code. To avoid end users to use the snapshot version, we have created 'release' branch for these repositories and make `release` branch as default branch. We are going to sync the code from the master branch to release after each release. 

### Version match regex update

With -SNAPSHOT as part of the version, the old way of regex matching is not working anymore for the [version-upgrade][] task. 

We have change regex from 


```
<version.light[a-z-]+4j>\d*\.\d*\.\d*</version.light[a-z-]+4j>
```

to 

```
<version.light-4j>((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)</version.light-4j>
```


### Gradle version upgrade

With some of our repositories using the Gradle build system, we need to cover it for the version upgrade. The light-4j version is defined in the `gradle.properties` file in the project home directory. 

The following is the regex for matching.

```
light4jVersion=((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)
```

### Release Snapshot Task

For the normal release, there are a lot of extra steps. For example, generate changelog and check in, push the release notes to the GitHub.com, etc. We don't need these steps for the snapshot release. To avoid updating these flags for back and force, it makes sense to create a branch new task in the light-bot release-maven folder called release-snapshot. 

The config files are similar to the release-all, but a lot of steps are skipped. 

### Summary

With the above changes, we have successfully released the snapshot version to the maven central. You can view the snapshot jars from the following link. 

https://oss.sonatype.org/content/repositories/snapshots/com/networknt/body/1.6.1-SNAPSHOT/

At the moment, we are going to release the snapshot version on a regular basis manually. Soon, we will create a cron job to release nightly. 

[develop-build]: /tool/light-bot/task/develop-build/
[version-upgrade]: /tool/light-bot/task/version-upgrade/
