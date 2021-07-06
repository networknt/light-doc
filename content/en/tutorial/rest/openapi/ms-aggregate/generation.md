---
title: "Generation"
date: 2018-03-11T18:00:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Now we have four openapi.yaml files and four openapi.json files available. Let's use light-codegen
to start four projects in light-example-4j/rest/openapi/ms_aggregate. In a normal API build, you 
should create a repo for each API. For us, we have to user light-example-4j for all the
examples and tutorial for easy management in networknt github organization.

### Build light-codege

The project is cloned to the local already during the preparation stage. Let's build it.

```
cd ~/networknt/light-codegen
mvn clean install -DskipTests
```

### Prepare Generator Config

Each generator in light-codegen requires several parameters to control how the generator
works. For more information on how to use generator, please refer to [light-codegen][].

For API AA, here is the config.json and a copy can be found in the folder 
model-config/rest/openapi/aa/1.0.0 along with openapi.yaml and openapi.json. 

```json
{
  "name": "aa",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "aa",
  "rootPackage": "com.networknt.aa",
  "handlerPackage":"com.networknt.aa.handler",
  "modelPackage":"com.networknt.aa.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 7001,
  "enableHttp": false,
  "httpsPort": 7441,
  "enableHttps": true,
  "enableRegistry": false,
  "supportDb": false,
  "supportH2ForTest": false,
  "supportClient": true
}
```

As you can see the generated project will use 7441 for https and client module will be 
included as it will call API AB, AC, AD with it. DB dependencies are not required for
this tutorial.


### Generate first project

Now you have your light-codegen built, let's generate a project. Assume that
model-config, light-example-4j and light-codegen are in the same working
directory ~/networknt and you are in ~/networknt now.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/ms-aggregate/aa/generated -m model-config/rest/openapi/aa/1.0.0/openapi.json -c model-config/rest/openapi/aa/1.0.0/config.json
```

### Build and run the mock API

And now you have a new project created in light-example-4j/rest/openapi/ms-aggregate/aa/generated. 
Let's build it and run the test cases. If everything is OK, start the server.

```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/aa/generated
mvn clean install exec:exec
```

Let's test the API AA by issuing the following command
```
curl -k https://localhost:7441/v1/data
```

By default the generated response example will be returned. 

```
 ["Message 1","Message 2"]
```

### Generate other APIs

Let's kill the API AA by Ctrl+C and move to the light-codegen terminal again. Follow 
the above steps to generate other APIs. Make sure you are in ~/networknt directory.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/ms-aggregate/ab/generated -m model-config/rest/openapi/ab/1.0.0/openapi.json -c model-config/rest/openapi/ab/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/ms-aggregate/ac/generated -m model-config/rest/openapi/ac/1.0.0/openapi.json -c model-config/rest/openapi/ac/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/ms-aggregate/ad/generated -m model-config/rest/openapi/ad/1.0.0/openapi.json -c model-config/rest/openapi/ad/1.0.0/config.json
```

Now you have four APIs generated from four OpenAPI specifications. Let's check
them in. Note that you might not have write access to this repo, so you can ignore
this step. 

```
cd ~/networknt/light-example-4j
git add .
git commit -m "checkin 4 apis for OpenAPI ms-aggregate"
git push origin master
```

[light-codegen]: /tutorial/generator/openapi/
