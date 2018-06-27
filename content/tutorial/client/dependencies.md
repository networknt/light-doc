---
title: Including Client Dependencies
linktitle: Including Client Dependencies 
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: "How to include the client dependencies in your application for light-4j API consumption."
weight: 10
sections_weight: 10
draft: false
toc: true
---

This dependency will include the submodules required to support client consumption of an api like service discovery,
load balancing, oauth2 integration, etc..

## Maven

```xml
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>light-client-all-4j</artifactId>
    <version>1.5.15</version>
</dependency>
```

## Gradle

```groovy
dependencies {
    compile 'com.networknt:light-client-all-4j:1.5.15'
}
```




