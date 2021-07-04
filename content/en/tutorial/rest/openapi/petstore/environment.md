---
title: "Environment"
date: 2017-11-08T10:54:56-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 30
aliases: []
toc: false
draft: false
reviewed: true
---

You will need the following dependencies before starting this tutorial:

- Java JDK 11 (I prefer OpenJDK but Oracle JDK will do)
- Maven
- Git
- Docker

The use of docker-compose is optional but can sometimes 
be problematic for Windows users. 
 
Assuming the above software packages are installed, we need to create a workspace and clone the 
projects needed for this tutorial. The following will assume your workspace is in a directory 
named networknt under your home directory. 

```
cd ~
mkdir networknt
cd networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-oauth2.git
git clone https://github.com/networknt/light-docker.git
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-config-test.git
```

We are going to re-generate the petstore project in light-example-4j. Rename the existing directory 
in light-example-4j to petstore.bak

If you feel like you may have made an error while following this tutorial, you can compare your 
working code with the petstore.bak directory to find any differences.

```
cd ~/networknt/light-example-4j/rest/openapi
mv petstore petstore.bak
cd ~/networknt
```

Build the light-codegen project so we can use it to generate our petstore project. There are many ways 
to [use light-codegen][] but for the purposes of this tutorial, we will just build and use the codgen-cli.

```
cd ~/networknt/light-codegen
mvn install -DskipTests
```

Our environment and light-codegen are now ready. In the next step, we will [generate][] the project using 
the OpenAPI 3.0 specification. 

[generate]: /tutorial/rest/openapi/petstore/generate/
[use light-codegen]: /tutorial/generator/
