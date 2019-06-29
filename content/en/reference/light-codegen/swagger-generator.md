---
title: "Swagger Generator"
date: 2019-01-05T11:27:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This generator is based on Swagger 2.0 specification and it is the most popular specification
format for Restful API these days. Although it might be replaced soon with OpenAPI 3.0, this
is still the main style in light-rest-4j framework. A lot of our customers are using it to 
build their restful APIs. In light-rest-4j, we use the specification to scaffold the project
and we also use the specification during runtime for OAuth 2.0 scope verification and validation
against Swagger for Path Parameters, Query Parameters, Headers and Request Body if exists.


## Input

#### Model

In light-rest-4j framework generator, the model that drives code generation is the Swagger 2.0 
specification. When editing it, normally it will be in yaml format with separate files for 
readability and flexibility. Before leverage it in light-rest-4j framework, all yaml files need 
to be bundled and converted into json format in order to be consumed by the framework and generator. 
Also, a validation needs to be done to make sure that the generated swagger.json is valid against 
json schema of Swagger 2.0 specification. 
 
Note: currently, we support Swagger 2.0 specification and OpenAPI 3.0 specification. You can
find a similar [tutorial for OpenAPI 3.0 generator][]


- [Swagger Editor][]

- [Swagger CLI][]


#### Config

| **Field Name** | **Description** |
|------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| **name** | used in generated pom.xml for project name |
| **version** | used in generated pom.xml for project vesion |
| **groupId** | used in generated pom.xml for project groupId |
| **artifactId** | used in generated pom.xml for project artifactId |
| **rootPackage** | the root package name for your project and it will normally be your domain plug project name. |
| **handlerPackage** | the Java package for all generated handlers. |
| **modelPackage** | the Java package for all generated models or POJOs. |
| **overwriteHandler** | controls if you want to overwrite handler when regenerate the same project into the same folder. If you only want to upgrade the framework to another minor version and don't want to overwrite handlers, then set this property to false. |
| **overwriteHandlerTest** | controls if you want to overwrite handler test cases. |
| **overwriteModel** | controls if you want to overwrite generated models. |
| **httpPort** | is the port number of Http listener if enableHttp is true |
| **enableHttp** | specify if the server listens to http port. Http should only be enabled in dev. |
| **httpsPort** | the port number of Https listener if enableHttps is true. |
| **enableHttps** | specify if the server listens to https port. Https should be used in any official environment for security reason. |
| **enableRegistry** | control if built-in service registry/discovery is used. Only necessary if running as standalone java -jar xxx. |
| **enableParamDescription** | decide if generate parameter description from specifications as annotation. |
| **supportDb** | control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml |
| **dbInfo** | the database connection pool configuration info. |
| **supportH2ForTest** | if true, add H2 in pom.xml as test scope to support unit test with H2 database. |
| **supportClient** | if true, add com.networknt.client module to pom.xml to support service to service call. |

Here is an exmaple of config.json for swagger generator.

```json
{
  "name": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 8080,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "enableParamDescription": true,
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


In most of the cases, developers will only update handlers, handler tests and models in a project.
Of course, you might need different database for your project and we have a [database tutorial] that
can help you to further config Oracle and Postgres.   

Given we have most of our model and config files in model-config repo, most generator input would
from the rest folder in model-config for light-rest-4j framework.

Let's clone the project to your workspace as we will need it in the following steps.
I am using ~/networknt as workspace but it can be anywhere in your home directory.  


```
cd ~/networknt
git clone git@github.com:networknt/model-config.git
```

## Usage

#### Java Command line

Before using the command line to generate the code, you need to check out the repo and build it.

```
cd ~/networknt
git clone git@github.com:networknt/light-codegen.git
cd light-codegen
mvn clean install
```

Given we have test swagger.json and config.json in light-rest-4j/src/test/resources folder,
the following command line will generate a RESTful petstore API at /tmp/swagger-petstore folder. 

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f swagger -o /tmp/swagger-petstore -m light-rest-4j/src/test/resources/swagger.json -c light-rest-4j/src/test/resources/config.json
```
 
After you run the above command, you can build and start the service:

```
cd /tmp/swagger-petstore
mvn clean install exec:exec
```

To test the service from another terminal:
```
curl http://localhost:8080/v2/pet/111
```

The above example use local swagger specification and config file. Let's try to use files from
github.com:

Working directory: light-codegen

```
rm -rf /tmp/swagger-petstore
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f swagger -o /tmp/swagger-petstore -m https://raw.githubusercontent.com/networknt/model-config/master/rest/swagger/petstore/2.0.0/swagger.json -c https://raw.githubusercontent.com/networknt/model-config/master/rest/swagger/petstore/2.0.0/config.json
```

Please note that you need to use a raw url when accessing github files. The above command line will
generate a petstore service in /tmp/swagger-petstore.

Here is the example to generate petstore. Assuming model-config is in the same workspace as 
light-codegen.


Working directory: light-codegen

```
rm -rf /tmp/swagger-petstore
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f swagger -o /tmp/swagger-petstore -m model-config/rest/swagger/petstore/2.0.0/swagger.json -c model-config/rest/swagger/petstore/2.0.0/config.json

```

#### Docker Command Line

Above local build and command line utility works but it is very hard to use that in devops script. 
In order to make scripting easier, we have dockerized the command line utility. 


The following command is using docker image to generate the code into /tmp/light-codegen/generated:
```
docker run -it -v ~/networknt/light-codegen/light-rest-4j/src/test/resources:/light-api/input -v /tmp/light-codegen:/light-api/out networknt/light-codegen -f swagger -m /light-api/input/swagger.json -c /light-api/input/config.json -o /light-api/out/generated
```
On Linux environment, the generated code might belong to root:root and you need to change the
owner to yourself before building it.

```
cd /tmp/light-codegen
sudo chown -R steve:steve generated
cd generated
mvn clean install exec:exec
```
To test it.
```
curl localhost:8080/v2/pet/111
```

#### Output
Below is the folder structure after generate. 
```
.  
├── app-name
│   ├── LICENSE
│   ├── README.md
│   ├── build.sh
│   ├── dependency-reduced-pom.xml
│   ├── docker
│   │   ├── Dockerfile
│   │   └── Dockerfile-Redhat
│   ├── pom.xml
│   ├── app-name.iml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── networknt
│       │   │           └── app-name
│       │   │               ├── handler
│       │   │               │   ├── ItemDeleteHandler.java
│       │   │               │   ├── ItemGetHandler.java
│       │   │               │   ├── ItemPutHandler.java
│       │   │               │   └── ItemPostHandler.java
│       │   │               └── model
│       │   │                   └── Model.java
│       │   └── resources
│       │       ├── config
│       │       │   ├── handler.yml
│       │       │   ├── mask.yml
│       │       │   ├── openapi-security.yml
│       │       │   ├── openapi-validator.yml
│       │       │   ├── openapi.yaml
│       │       │   ├── primary.crt
│       │       │   ├── secondary.crt
│       │       │   ├── secret.yml
│       │       │   ├── server.keystore
│       │       │   ├── server.truststore
│       │       │   ├── server.yml
│       │       │   └── service.yml
│       │       ├── logback.xml
│       │       └── todos.properties
│       └── test
│           ├── java
│           │   └── com
│           │       └── networknt
│           │           └── app-name
│           │               └── handler
│           └── resources
│               ├── config
│               │   ├── client.keystore
│               │   ├── client.truststore
│               │   ├── client.yml
│               │   └── server.yml
│               └── logback-test.xml
└── app-name.iml
```

#### Reference of Generated Configs
[handler.yml](/configs/handler/)  
[mask.yml](/configs/mask/)    
openapi-security.yml  
openapi-validator.yml  
openapi.yaml  
primary.crt  
secondary.crt  
secret.yml  
server.keystore  
server.truststore  
server.yml  
service.yml  

#### Docker Scripting

You can use docker run command to call the generator but it is very complicated for the parameters.
In order to make things easier and friendlier to devops flow. Let's create a script to call the
command line from docker image.

If you look at the docker run command you can see that we basically need one input folder for 
schema and config files and one output folder to generated code. Once these volumes are mapped to 
local directory and with framework specified, it is easy to derive other files based on convention. 


```
cd ~/networknt/model-config
./generate.sh swagger ~/networknt/model-config/rest/swagger/petstore/2.0.0 /tmp/petstore
```
Now you should have a project generated in /tmp/petstore/genereted

#### Codegen Site

The service API is ready. We are working on the UI with a generation wizard.
 

[tutorial for OpenAPI 3.0 generator]: /tutorial/generator/openapi/
[Swagger Editor]: /tool/swagger-editor/
[Swagger CLI]: /tool/swagger-cli/
[database tutorial]: /tutorial/rest/swagger/database/

