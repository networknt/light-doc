---
title: "Build Systems and Dependency Management"
date: 2019-03-09T16:29:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

It is strongly recommended that you choose a build system that supports dependency management and that can consume artifacts published to the “Maven Central” repository. We would recommend that you choose Maven or Gradle. It is possible to get Light-4j to work with other build systems (Ant, for example), but they are not well supported.


### Dependency Management

Each release of Light-4j provides a list of dependencies that it supports. In practice, you do not need to provide a version for any of these dependencies in your build configuration but only light-4j version. When you upgrade Light-4j itself, these dependencies are upgraded as well in a consistent way. 

As light-4j trying to limit the size of the final deliverable to the smallest, it won't automatically include all light-4j middleware handlers. The users will need to pick up handlers they need and all other dependencies each individual handler depends on will be loaded automatically. 

Although light-4j middleware handlers are scattered in different repositories, they all share the same version as light-4j and same group `com.networknt`. In your build system, it would be very easy to specify a light-4j version to control all middleware handlers you want to include. 


### Maven

The following is the property that defined the light-4j version:

    <properties>
        <version.light-4j>1.5.30</version.light-4j>
    </properties>    


The following is a list of light-4j components that you need in your application: 


```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>config</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>utility</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>security</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>client</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

There is no need to specify the transitive dependencies in the pom.xml file. For example, there are some other libraries that the security module depends on, but there is no need to specify here. 


### Gradle

Light-4j supports Gradle with Kotlin DSL for some of the projects and Kotlin applications. We are working on the light-codegen to support Gradle along with Maven, and hopefully, users can choose between Maven and Gradle in the light-codegen cnfig.json file. 

The light-4j version is defined in the `gradle.properties` file.

```
light4jVersion=1.5.32
```

The dependencies are defined in the `build.gradle.kts` file.

```
    // light-4j
	val light4jVersion: String by project
    compile("com.networknt", "server", light4jVersion)
    compile("com.networknt", "handler", light4jVersion)
    compile("com.networknt", "info", light4jVersion)
    compile("com.networknt", "health", light4jVersion)
```

