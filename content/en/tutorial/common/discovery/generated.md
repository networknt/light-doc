---
title: Introduction and Code Generation
linktitle: Introduction and Code Generation
description: "The starting point of the discovery tutorials will walk you through
code generation for the services that will be used throughout."
date: 2017-10-17T17:58:29-04:00
lastmod: 2018-05-15
weight: 10
sections_weight: 10
draft: false
toc: true
reviewed: true
---

### Introduction

This is a tutorial to show you how to use service registry and discovery for microservices. The example services are implemented in a RESTful style but can be implemented in GraphQL or Hybrid as well. We are going to use api_a, api_b, api_c, and api_d as our examples, with security disabled for simplicity. There are some details might not be shown in this tutorial,  for example, walking through light-codegen config files, etc. It is recommended to go through [ms-chain][] before this tutorial. 

### Preparation

In order to follow the steps below, please make sure you have your [development environment][] setup.

* Create a working directory under your user directory called networknt.

```bash
mkdir ~/networknt
```

### Clone the specifications

In order to generate the initial projects, we use light-codegen to scaffold these services from configuration outlining the API specs.

```bash
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```

### Code generation

In order to generate the four projects, let's clone and build the light-codegen in the workspace. We are going to use the command line utility instead of the Docker container.

```bash
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install
```

We are going to generate these preject into the light-example-4j repository. Let's clone it and move the discovery folder to discovery.bak so that you can compare your work with the checked in copy. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j
mv discovery dicovery.bak
```

Now let's generate the four APIs.

```bash
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/discovery/api_a/generated -m model-config/rest/openapi/aa/1.0.0/openapi.yaml -c model-config/rest/openapi/aa/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/discovery/api_b/generated -m model-config/rest/openapi/ab/1.0.0/openapi.yaml -c model-config/rest/openapi/ab/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/discovery/api_c/generated -m model-config/rest/openapi/ac/1.0.0/openapi.yaml -c model-config/rest/openapi/ac/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/discovery/api_d/generated -m model-config/rest/openapi/ad/1.0.0/openapi.yaml -c model-config/rest/openapi/ad/1.0.0/config.json

```

We have four projects generated in `~/networknt/light-example-4j` within the discovery folder. 

### Test generated code

Now you can test the generated projects to make sure they are working with mock data. We will pick up one project to test it, but you can test them all.

```bash
cd ~/networknt/discovery/api_a/generated
mvn clean install exec:exec
```

From another terminal, access the server with curl command and check the result.

```bash
curl -k https://localhost:7441/v1/data
["Message 1","Message 2"]
```

Based on the config files for each project, the generated service will listen to 7441, 7442, 7443 and 7444 for https connections. You can start other services to test them on their corresponding port with the same path.

This concludes the first step, and we now have four generated APIs in working condition. In the next step, we are going to change it to enable API to API communication.

Next Step: [static][]

[ms-chain]: /tutorial/rest/swagger/ms-chain/
[static]: /tutorial/common/discovery/static/
[development environment]: /development/develop-build/
