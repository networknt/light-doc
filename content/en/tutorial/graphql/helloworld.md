---
title: "Hello World Tutorial"
date: 2018-01-03T10:31:15-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Like [light-rest-4j][] we have light-codegen to generate a project from either Swagger 2.0 or OpenAPI 3.0 specification, we can generate GraphQL project from GraphQL IDL. Or generate without providing any model. For this Hello World example, we are going to generate it with only a config.json. 

### Prepare environment

First, we need to clone light-codegen, model-config, and light-example-4j into your workspace. I am using networknt as workspace in my home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
``` 

Let's build light-codegen so that we can use the command line to generate the code.

```
cd ~/networknt/light-codegen
mvn clean install -DskipTests
```

### Config

Let's take a look at the config.json that control how the project is going to be generated. It can be found at https://github.com/networknt/model-config/blob/master/graphql/helloworld/config.json

Here is the content.


```json
{
  "name": "starwars",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "starwars",
  "schemaPackage": "com.networknt.starwars.schema",
  "schemaClass": "StarWarsSchema",
  "overwriteSchemaClass": true,
  "httpPort": 8080,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false
}
```

### Generate

Given the light-codegen is built, let's generate the project from the config file only without IDL. For more detail about the light-codegen please refer to [light-codegen graphql][].

We are going to generate the code to the same folder on light-example-4j/graphql/helloworld, let's rename the existing folder so that you can compare after you follow the tutorial. 

```
cd ~/networknt/light-example-4j/graphql
mv helloworld helloworld.bak
```

Now let's run the command line in `~/networknt` folder.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-graphql-4j -o light-example-4j/graphql/helloworld -c model-config/graphql/helloworld/config.json

```

### Build and Start

Now let's build and start the server. 

```
cd ~/networknt/light-example-4j/graphql/helloworld
mvn clean install exec:exec

```


### Test

To test the server: 

```
curl -k -H 'Content-Type:application/json' -X POST https://localhost:8443/graphql -d '{"query":"{ hello }"}'
```

and the result is:

```
{"data":{"hello":"world"}}
```

To access GraphiQL, you need to put the following url into your browser's address.

```
https://localhost:8443/graphql
```

On the left panel, enter the following to test the result.
 
```
{ hello }
```

Result

```
{
  "data": {
    "hello": "world"
  }
}
```

### Rest Endpoints

With the handler.yml configuration for the middleware handler chain, we can add some Rest handlers to the mix so that the same server can serve both GraphQL and Restful endpoints. 

In the generated GraphQL project, we have server/info and health endpoints built in. To access these endpoint. 

```
curl -k https://localhost:8443//health/com.networknt.starwars-1.0.1
```

And the result is 

```
OK
```

```
curl -k https://localhost:8443/server/info
```

And the result is 

```
{"deployment":{"apiVersion":"SNAPSHOT","frameworkVersion":"2.0.5-SNAPSHOT"},"environment":{"host":{"ip":"127.0.0.1","hostname":"freedom","dns":"localhost"},"runtime":{"availableProcessors":16,"freeMemory":485311088,"totalMemory":528482304,"maxMemory":8432648192},"system":{"javaVendor":"Azul Systems, Inc.","javaVersion":"11.0.3","osName":"Linux","osVersion":"4.15.0-58-generic","userTimezone":"America/Toronto"}},"security":{"serverFingerPrint":"873b92f50afe99e9c5af38f1c865987ac113191d"},"specification":null,"component":{"com.networknt.correlation.CorrelationHandler":{"enabled":true,"autogenCorrelationID":true},"com.networknt.graphql.validator.ValidatorHandler":{},"com.networknt.traceability.TraceabilityHandler":{"enabled":true},"com.networknt.exception.ExceptionHandler":{"description":"Exception handler for runtime exception and ApiException if it is not handled by other handlers in the chain","enabled":true}}
```



[light-rest-4j]: /style/light-rest-4j/
[light-codegen graphql]: /tutorial/generator/graphql/
