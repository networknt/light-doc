---
title: "Generation"
date: 2017-11-29T14:42:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Now that we have four API swagger.yaml files available, let's use light-codegen
to start four projects in light-example-4j/rest/swagger/ms_chain. In a normal API build, 
you should create a repo for each API. For us, we have to user light-example-4j for all 
the examples and tutorial for easy management in networknt github organization.

#### Build light-codegen

The light-codegen project is cloned to the local already during the prepare stage. Let's 
build it.

```
cd ~/networknt/light-codegen
mvn clean install -DskipTests
```

#### Prepare Generator Config

Each generator in light-codegen requires several parameters to control how the generator
works. For more information on how to use generator, please refer [light-codegen][]

For API A, here is the config.json and a copy can be found in the folder 
model-config/rest/swagger/api_a/1.0.0 along with swagger.yaml and swagger.json. 

```
{
  "name": "apia",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "apia",
  "rootPackage": "com.networknt.apia",
  "handlerPackage":"com.networknt.apia.handler",
  "modelPackage":"com.networknt.apia.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 7001,
  "enableHttp": true,
  "httpsPort": 7441,
  "enableHttps": true,
  "enableRegistry": false,
  "supportDb": false,
  "supportH2ForTest": false,
  "supportClient": true
}
```
As you can see the generated project will use 7001 for http and 7441 for https and client 
module will be included as it will call API B with it. DB dependencies are not required for 
this tutorial.


#### Generate first project

Now you have your light-codegen built, let's generate a project. Assume that model-config, 
light-example-4j and light-codegen are in the same working directory ~/networknt and you are 
in ~/networknt folder now.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f swagger -o light-example-4j/rest/swagger/ms_chain/api_a/generated -m model-config/rest/swagger/api_a/1.0.0/swagger.json -c model-config/rest/swagger/api_a/1.0.0/config.json
```

You might realized that light-codegen is very fast compared to other code generators. It is 
due to it using a rocker template engine which compiles templates into Java classes during the build 
process. It supports dynamic template reloading but we are not using it. That is why is has
a info logging "INFO com.fizzed.rocker.runtime.RockerRuntime - Rocker template reloading not 
activated". This message can be simply ignored. 


#### Build and run the mock API

And now you have a new project created in light-example-4j/rest/swagger/ms_chain/api_a/generated. 
Let's build it and run the test cases. If everything is OK, start the server.

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/generated
mvn clean install exec:exec
```

Let's test the API A by issuing the following command
```
curl localhost:7001/v1/data
```

By default the generated response example will be returned. 

```
 ["Message 1","Message 2"]
```

As https port is enable, let's test https connection and the result should be the same.

```
curl -k https://localhost:7441/v1/data
```


#### Generate other APIs

Let's kill the API A by Ctrl+C and move to the workspace ~/networknt again. Follow the above steps 
to generate other APIs. Make sure you are in ~/networknt directory.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f swagger -o light-example-4j/rest/swagger/ms_chain/api_b/generated -m model-config/rest/swagger/api_b/1.0.0/swagger.json -c model-config/rest/swagger/api_b/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f swagger -o light-example-4j/rest/swagger/ms_chain/api_c/generated -m model-config/rest/swagger/api_c/1.0.0/swagger.json -c model-config/rest/swagger/api_c/1.0.0/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f swagger -o light-example-4j/rest/swagger/ms_chain/api_d/generated -m model-config/rest/swagger/api_d/1.0.0/swagger.json -c model-config/rest/swagger/api_d/1.0.0/config.json
```

Now you have four APIs generated from four OpenAPI specifications. Let's check
them in. Note that you might not have write access to this repo, so you can ignore
this step. 

```
cd ~/networknt/light-example-4j
git add .
git commit -m "checkin 4 apis"
git push origin master
```


[light-codegen]: /tool/light-codegen/
