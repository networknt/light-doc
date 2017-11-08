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
---


You need to have Java JDK 8 (I prefer OpenJDK but Oracle JDK will do), Maven, Git and 
Docker installed before starting this tutorial.
 
Assuming above software packages are installed, let's create a workspace and clone the 
projects we need for the tutorial. The following will assume to your workspace is named
networknt under your home directory. 

```
cd ~C
mkdir networknt
cd networknt
git clone git@github.com:networknt/light-codegen.git
git clone git@github.com:networknt/light-example-4j.git
git clone git@github.com:networknt/light-oauth2.git
git clone git@github.com:networknt/light-docker.git
git clone git@github.com:networknt/model-config.git
```

We are going to re-generate petstore project in light-example-4j. So let's rename
the existing directory in light-example-4j to petstore.bak

```
cd ~/networknt/light-example-4j/rest
mv petstore petstore.bak
cd ~/networknt
```

Now let's build the light-codegen to make it ready to generate petstore project. There are
many ways to use light-codegen but here we just build and use the command line codgen-cli.

```
cd ~/networknt/light-codegen
mvn install -DskipTests
```

Now, we have the environment and light-codegen ready. The next step, we will generate the
project with the OpenAPI specification. 
