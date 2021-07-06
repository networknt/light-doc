---
title: "Dockerfile"
date: 2019-06-28T21:27:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Light-codegen generates a docker folder in the root of the target project, and this folder contains several Dockerfiles to build different Docker images. 

For 1.6.x Java 8 branch, we have a default Dockerfile based on the Alpine image and Dockerfile-RedHat based on the RedHat image for some enterprise customers. 

When we move to Java 11, we have added several more Dockerfiles.

### Dockerfile

```
FROM openjdk:11.0.3-slim
ADD /target/light-router.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```
It is the default Dockerfile based on openjdk:11.0.3-slim. It is several hundreds of MB in size but it is official image. 

### Dockerfile-Debug

```
FROM openjdk:11.0.3-slim
ADD /target/light-router.jar server.jar
CMD ["/bin/sh","-c","java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

This is very similar to the default one with `agentlib:jdwp` added to the command line. It supports the [remote debugging][] from the IDE. 

### Dockerfile-RedHat

We still want to support RedHat JDK 11 but currently, we only have a placeholder here. 

### Dockerfile-Zulu

```
FROM azul/zulu-openjdk-alpine:11 as packager

RUN { \
        java --version ; \
        echo "jlink version:" && \
        jlink --version ; \
    }

ENV JAVA_MINIMAL=/opt/jre

# build modules distribution
RUN jlink \
    --verbose \
    --add-modules \
        java.base,java.sql,java.naming,java.desktop,java.xml,jdk.crypto.cryptoki \
    --compress 2 \
    --strip-debug \
    --no-header-files \
    --no-man-pages \
    --output "$JAVA_MINIMAL"

# Second stage, add only our minimal "JRE" distr and our app
FROM alpine

ENV JAVA_MINIMAL=/opt/jre
ENV PATH="$PATH:$JAVA_MINIMAL/bin"

COPY --from=packager "$JAVA_MINIMAL" "$JAVA_MINIMAL"
COPY /target/light-router.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]

```

This is the smallest image, and the final size for the light-router is just about 75MB. It is based on the Alpine Linux and uses Jlink to include only the modules that light-4j needs. 


[remote debugging]: /tutorial/common/debug/docker-remote/
