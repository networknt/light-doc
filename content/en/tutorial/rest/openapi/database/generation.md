---
title: "Generation"
date: 2017-11-24T08:21:04-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 30      #rem
aliases: []
toc: false
draft: false
reviewed: true
---

With the specification in place, we can generate the code with [light-codegen][]

There are three different ways to generate the code with light-codegen:

* Local build
* Docker container
* Script with docker container

To learn how to use the tool, please refer to the [light-codegen tutorial][]

### Generate code with local build

Clone and build light-codegen

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install -DskipTests
```

For this demo, I am going to generate the code into light-example-4j/rest/openapi/database/generated folder so that users can check the code later on from this repository. 

Let's check out the light-example-4j repository and back up the existing database project. You can later compare your working folder with the .bak folder to see any difference. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j/rest/openapi
mv database database.bak
```

Before generating the project, we need to create a config.json to define packages, artifactId, groupId, and other options for the project.

Here is the content of the file and it can be found in `~/networknt/model-config/rest/openapi/database`

```
{
  "name": "database",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "database",
  "rootPackage": "com.networknt.database",
  "handlerPackage":"com.networknt.database.handler",
  "modelPackage":"com.networknt.database.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": true,
  "supportClient": false,
  "dockerOrganization": "networknt"
}
```

As you can see, we have enabled the Mysql database and H2 for test cases. 

Code generation

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/database/generated -m model-config/rest/openapi/database/openapi.yaml -c model-config/rest/openapi/database/config.json
```

Now you should have a project generated. Let's build it and run it.

```
cd ~/networknt/light-example-4j/rest/openapi/database/generated
mvn clean install exec:exec
```

Now you can access the service with curl following the [next step][] below.


### Generate code with docker container

Let's remove the generated folder from light-example-4j/rest/openapi/database folder and
generate the project again with the docker container.

```
cd ~/networknt/light-example-4j/openapi/rest/database
rm -rf generated
```

Now let's generate the project again with docker.

```
cd ~/networknt
docker run -it -v ~/networknt/model-config/rest/openapi/database:/light-api/input -v ~/networknt/light-example-4j/rest/openapi/database:/light-api/out networknt/light-codegen -f openapi -m /light-api/input/openapi.yaml -c /light-api/input/config.json -o /light-api/out/generated
```

Note that on Linux, the generated folder in ~/networknt/light-example-4j/rest/openapi/database will be owned by root:root and it needs to be changed to your user:group before compiling the project. On other operating systems like Mac and Windows, you don't need this step. 

```
cd ~/networknt/light-example-4j/rest/openapi/database
sudo chown -R steve:steve .
```

Let's build and start the service

```
cd ~/networknt/light-example-4j/rest/openapi/database/generated
mvn clean install exec:exec
```

Now you can access the service with curl following the [next step][].

[light-codegen]: /tool/light-codegen/
[light-codegen tutorial]: /tutorial/generator/
[next step]: /tutorial/rest/openapi/database/test/

