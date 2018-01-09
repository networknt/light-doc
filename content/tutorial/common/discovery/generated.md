---
date: 2017-10-17T17:58:29-04:00
title: Introduction and code generation
---

# Introduction

This is a tutorial to show you how to use service registry and discovery
for microservices. The example services are implemented in RESTful style but
they can be implemented in graphql or hybrid as well. We are going to use 
api_a, api_b, api_c and api_d as our examples. To simply the tutorial, I am 
going to disable the security all the time. There are some details that might
not be shown in this tutorial, for example, walking through light-codegen config
files etc. It is recommended to go through [ms-chain](ms-chain/) before this
tutorial. 

There are two situations that you need service discovery: 

* A client application to consume services and it needs to locate the service
instance by serviceId

* A service that has to call another service to fulfill its request in request/
response style.  

The specifications for above APIs can be found at 
https://github.com/networknt/model-config rest folder.

# Preparation

In order to follow the steps below, please make sure you have the same 
working environment.

* A computer with MacOS or Linux (Windows should work but I never tried)
* Install git
* Install Docker
* Install JDK 8 and Maven
* Install Java IDE (Intellij IDEA Community Edition is recommended)
* Create a working directory under your user directory called networknt.

```
cd ~
mkdir networknt
```

# Clone the specifications

In order to generate the initial projects, we need to call light-codegen
and we need the specifications for these services.

```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```

In this repo, you have a generate.sh in the root folder to use docker
container of light-codegen to generate the code and there are api_a,
api_b, api_c and api_d in rest folder for swagger.json files and config.json
files for each API.

# Code generation

We are going to generate the code into light-example-4j repo, so let's
clone this repo into our working directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
```

In the above repo, there is a folder discovery contains all the projects
for this tutorial. In order to start from scratch, let's change the existing
folder to discovery.bak as a backup so that you can compare if your code is
not working in each step.

```
cd ~/networknt/light-example-4j
mv discovery discovery.bak
```

In order to generate the four projects, let's clone and build the light-codegen 
into the workspace. 

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
mvn clean install
```

Now let's generate the four APIs.

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f light-rest-4j -o ../light-example-4j/discovery/api_a/generated -m ../model-config/rest/api_a/1.0.0/swagger.json -c ../model-config/rest/api_a/1.0.0/config.json
java -jar codegen-cli/target/codegen-cli.jar -f light-rest-4j -o ../light-example-4j/discovery/api_b/generated -m ../model-config/rest/api_b/1.0.0/swagger.json -c ../model-config/rest/api_b/1.0.0/config.json
java -jar codegen-cli/target/codegen-cli.jar -f light-rest-4j -o ../light-example-4j/discovery/api_c/generated -m ../model-config/rest/api_c/1.0.0/swagger.json -c ../model-config/rest/api_c/1.0.0/config.json
java -jar codegen-cli/target/codegen-cli.jar -f light-rest-4j -o ../light-example-4j/discovery/api_d/generated -m ../model-config/rest/api_d/1.0.0/swagger.json -c ../model-config/rest/api_d/1.0.0/config.json

```

We have four projects generated and compiled under generated folder under each 
project folder. 

# Test generated code

Now you can test the generated projects to make sure they are working with mock
data. We will pick up one project to test it but you can test them all.

```
cd ~/networknt/light-example-4j/discovery/api_a/generated
mvn clean install exec:exec
```

From another terminal, access the server with curl command and check the result.

```
curl http://localhost:7001/v1/data
["Message 1","Message 2"]
```

Based on the config files for each project, the generated service will listen to
7001, 7002, 7003 and 7004 for http connections. You can start other services to test
them on their corresponding port.

This conclude the first step and we now have four generated APIs in working condition.
The next step, we are going to change it to use static configuration to enable API to
API communication.



 