---
title: "Eventuate Generator"
date: 2019-01-05T11:27:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

[Light-codegen][] provides a generator to generate eventuate base API based on OpenAPI 3.0 specification.

The eventuate based API is built based on light-eventuate-4j framework. light-eventuate-4j framework design eventuate microservice API based on CQRS (Command Query Responsibility Segregation) design pattern.
The service specification defined for Eventuate based API code generator should include the service definition for both CQRS command side service and query side service.

Command side service will publish event to event store to change the state of the domain object or entire system

Query side service will subscribe the event from event store and return results only and do not change the state of an object



## Input

#### Model

In light-eventuate-4j framework generator, the model that drives code generation is the OpenAPI
specification which same as normal light-rest-4j API OpenAPI specification.
But different as normal light-rest-4j api OpenAPI specification, the eventuate based rest API Json format specification includes two services module definition (command service & query service)

The base Json format will like the following:

```
{
"command":
{
...
}
"query":
{
...
}
}

```

User can refer to [todo-list example spec][] to build their own specification for  eventuate based rest API generation.




#### Config

| **Field Name** | **Description** |
|--------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| **name** | used in generated pom.xml for project version |
| **version** | used in generated pom.xml for project version |
| **groupID** | used in generated pom.xml for project groupId |
| **artifactId** | used in generated pom.xml for project artifactId |
| **rootPackage** | the root package name for your project and it will normally be your domain plug project name. |
| **handlerPackage** | the Java package for all generated handlers. |
| **modelPackage** | the Java package for all generated models or POJOs |
| **eventuateEventPackage** | the Java package for Eventuate common module which defined components could be shared by command service and query service; Normally we can put events objects and domain objects in this module. |
| **eventuateCommandPackage** | the Java package for Eventuate command side component module; Normally we put commands definition and aggregation classes in this module. |
| **eventuateQueryPackage** | the Java package for Eventuate query side component module; It includes query side DAO or repository service module. |
| **overwriteHandler** | controls if you want to overwrite handler when regenerate the same project into the same folder. If you only want to upgrade the framework to another minor version and don’t want to overwrite handlers, then set this property to false. |
| **overwriteHandlerTest** | control if you want to overwrite handler test cases |
| **overwriteModel** | controls if you want to overwrite generated models |
| **overwriteEventuateModule** | controls if you want to overwrite generated eventuate modules, which include commnon module, command module and query module. |
| **httpPort** | the port number of Http listener if enableHttp is `true`. |
| **enableHttp** | to specify if the server listens to http port. Http should only be enabled in dev. |
| **httpsPort** | the port number of Https listener if enableHttps is `true`. |
| **enableHttps** | to specify if the server listens to https port. Https should be used in any official environment for security reason. |
| **enableRegistry** | to control if built-in service registry/discovery is used. Only necessary if running as standalone java -jar xxx. |
| **supportDb** | to control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml |
| **dbInfo** | section is the database connection pool configuration info. |
| **supportH2ForTest** | if `true`, add H2 in pom.xml as test scope to support unit test with H2 database. |
| **supportClient** | if `true`, add com.networknt.client module to pom.xml to support service to service call. |

Here is an example of config.json for openapi generator.

```json
{
  "name": "todolist",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "todolist",
  "rootPackage": "com.networknt.todolist.rest",
  "handlerPackage":"com.networknt.todolist.rest.handler",
  "modelPackage":"com.networknt.todolist.rest.model",
  "eventuateEventPackage":"com.networknt.todolist.event",
  "eventuateCommandPackage":"com.networknt.todolist.command",
  "eventuateQueryPackage":"com.networknt.todolist.query",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": false,
  "overwriteEventuateModule": true,
  "httpPort": 8082,
  "enableHttp": true,
  "httpsPort": 8482,
  "enableHttps": true,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": true,
  "supportPostgresql": false,
  "supportH2ForTest": true,
  "supportClient": false,
  "dockerOrganization": "networknt"
}
```


For initialize the eventuate based the API service, the config value `overwriteEventuateModule` should be set as `true`.
The the light-codegen eventuate generator will generate the eventuate rest API project with the following modules:

-- **common module**:      Shared components module which will be used on both  command side service module and  query side service module. Normally, it includes events definition and data module for rest API

-- **command module**:     Eventuate command side component module; Normally we  put commands definition and aggregation classes in this module.

-- **query module**:       Eventuate query side component module; It includes query side DAO or repository service module.

-- **command side service module**:     CQRS command side service module

-- **query side service module**:       CQRS query side service module



For light-codegen eventuate generator, there is an example [todo-list spec & config for light-codegen][] in model-config repo.



Let's clone the project to your workspace as we will need it in the following steps.

Assume ~/networknt is the workspace root folder, but it can be anywhere in your home directory.


```
cd ~/networknt
git clone git@github.com:networknt/model-config.git
```


## Usage

#### Java Command line

You can [download or build][] the codegen-cli command line jar. 

Using the example spec and config in model-config repo, generate eventuate based API todo-list.

Working directory: ~/networknt

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f eventuate-rest -o /tmp/eventuate/todo-list -m model-config/rest/openapi/eventuate/todo-list/schema.json -c model-config/rest/openapi/eventuate/todo-list/config.json

```
 
After you run the above command, you can build and start the service:
```
cd /tmp/eventuate/todo-list
mvn clean install
```

Then user can start API developing the detail services from the generated project. For the detail development for the eventuate based API, please refer the [todo-list example][]  or [account-transfer example][] code on the light-example-4j repo

The [eventuate tutorial][] document includes the detail information about light-eventuate-4j framework and how to use it for eventuate based service API.


#### Codegen Site

You can generate single project or multiple projects from the site https://codegen.lightapi.net with your model and config files. 

[todo-list example spec]: https://github.com/networknt/model-config/blob/develop/rest/openapi/eventuate/todo-list/schema.json
[Light-codegen]: https://github.com/networknt/light-codegen
[eventuate tutorial]: /tutorial/eventuate/
[todo-list spec & config for light-codegen]: https://github.com/networknt/model-config/tree/develop/rest/openapi/eventuate/todo-list
[todo-list example]: https://github.com/networknt/light-example-4j/tree/master/eventuate/todo-list
[account-transfer example]: https://github.com/networknt/light-example-4j/tree/master/eventuate/account-management
[download or build]: /tool/light-codegen/download-build/
