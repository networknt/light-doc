---
title: "Memory Limit"
date: 2018-04-30T11:43:32-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "deployment"
    weight: 60
weight: 60
aliases: []
toc: false
draft: false
reviewed: true
---

### Java Heap Memory Limit in Docker

The issue is that if Xmx option is not defined explicitly, then JVM uses 1/4th of all memory available for the host OS due to a default internal garbage collection (GC) ergonomic algorithm. This can lead to killing the Java process by the kernel if JVM memory usage grows over the cgroups limit defined for a Docker container.

To solve this problem, one improvement was recently implemented in OpenJDK 9 and backported to OpenJDK 8 in version 8u131.  

https://github.com/docker-library/openjdk/pull/71
https://bugs.openjdk.java.net/browse/JDK-8175898
https://bugs.openjdk.java.net/browse/JDK-8170888
https://blog.csanchez.org/2017/05/31/running-a-jvm-in-a-container-without-getting-killed/

The command line to set it up is: 

```
java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm -version
```

As we are using later version of Java 8 on both Alpine and Redhat container, we are going to update the light-codegen to add these options in the java command line. Once this is done, we can control the container memory limitation with Kubernetes deployment configuration file. 

