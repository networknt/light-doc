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

You will need the following dependencies before starting this tutorial:

- Java JDK 8 (I prefer OpenJDK but Oracle JDK will do)
- Maven
- Git
- Docker

The use of docker-compose is optional but can sometimes 
be problematic for Windows users. 
 
Assuming above software packages are installed, let's create a workspace and clone the 
projects we need for the tutorial. The following will assume your workspace is named
networknt under your home directory. 

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

We are going to **re-generate** petstore project in light-example-4j. So let's rename
the existing directory in light-example-4j to petstore.bak

At any step, if you are unsure if you have followed correctly, you can always compare
your working code with petstore.bak folder to find the difference.

```
cd ~/networknt/light-example-4j/rest/swagger
mv petstore petstore.bak
cd ~/networknt
```

Now let's build the light-codegen to make it ready to generate petstore project. There are
many ways to [use light-codegen][] but here we just build and use the command line codgen-cli.

```
cd ~/networknt/light-codegen
mvn install -DskipTests
```

Now, we have the environment and light-codegen ready. The next step, we will [generate][] the
project with the Swagger 2.0 specification. 

[generate]: /tutorial/rest/swagger/petstore/generate/
[use light-codegen]: /tutorial/generator/
