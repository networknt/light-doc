---
title: "OpenAPI Generator"
date: 2019-01-05T11:27:08-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This generator is based on OpenAPI 3.0 specification, and it is a very new specification that is supposed to replace Swagger 2.0 specification. It has some significant changes to enhance the spec definition and simply the validate with only JSON schema. In my opinion, it is much easier to use, and the implementation is much simpler than Swagger 2.0.
OpenAPI adds a number of features; the only issue I can see is the entire toolchains around Swagger 2.0 are not migrated to OpenAPI 3.0 yet, and we have to build our own [openapi-parser][] for parsing and validation.

## Input

#### Model

In light-rest-4j framework generator, the model that drives code generation is the OpenAPI 3.0 specification previously named Swagger specification. When editing it, it usually will be in YAML format with separate files for readability and flexibility. Before leverage it in the light-rest-4j framework, all YAML files need to be bundled to a single file in YAML or JSON format to be consumed by the framework and generator. Also, validation needs to be done to make sure that the openapi.yaml or openapi.json is valid against JSON schema of OpenAPI 3.0 specification.

Note: currently, we support both OpenAPI 3.0 specification and Swagger 2.0 specification. There is a similar [Swagger 2.0 generator tutorial][] available.

- [Swagger Editor][]

- [Swagger CLI][]


#### Config

<!-- Reference Table -->

| Field Name | Description |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **name** | Used in generated prom.xml for project name <br><br> *Optionality*: mandatory |
| **version** | Used in generated pom.xml for project version<br><br>*Optionality*: mandatory |
| **groupId** | Used in generated pom.xml for project groupId<br><br> *Optionality*: mandatory |
| **artifactId** | used in generated pom.xml for project artifactId<br><br> *Optionality*: mandatory |
| **rootPackage** | root package name for your project and it will normally be your domain plug project name. <br>*Optionality*: optional |
| **handlerPackage** | the Java package for all generated handlers. <br><br>*Optionality*: mandatory |
| **modelPackage** | the Java package for all generated models or POJOs. <br><br>*Optionality*: mandatory |
| **overwriteHandler** | controls if you want to overwrite handler when regenerating the same project into the same folder. <br>*Optionality*: mandatory <br>Recommendation: set to `false` if you only want to upgrade the framework to another minor version and don't want to overwrite handlers |
| **overwriteHandlerTest** | controls if you want to overwrite handler test cases. <br><br>*Optionality*: mandatory |
| **overwriteModel** | controls if you want to overwrite generated models. <br><br>*Optionality*: mandatory Recommendation: set to `false` to prevent the model classes being overwritten |
| **generateModelOnly** | controls whether you wish to generate only the model classes <br><br>*Optionality*: optional <br>Recommendation: to be used by teams consuming an API and who wish to generate the model classes only Default: `false` |
| **regenerateCodeOnly** | controls whether you wish to regenerate only the model and handler classes, while skipping the underlying scripts, pom.xml and other files <br><br>*Optionality*: optional <br>Recommendation: to be used when there are changes in the model and teams wish to regenerate only artifacts affected by the change: model, handler classes, and handler.yml <br>Default: `false` |
| **generateValuesYml** | controls whether a values.yml is to be generated, with commonly changed values across test and production environments <br>Default: `false` |
| **httpPort** | the port number of Http listener if enableHttp is `true` <br><br>*Optionality*: mandatory |
| **enableHttp** | specify if the server listens to http port. <br><br>*Optionality*: mandatory <br>Recommendation: Http should be enabled in dev |
| **httpsPort** | the port number of Https listener if enableHttps is `true`. <br><br>*Optionality*: mandatory |
| **enableHttps** | specify if the server listens to https port. <br><br>*Optionality*: mandatory <br>Recommendation: Https should be used in any official environment for security reason Note: when Https is enabled, Http will automatically be disabled |
| **enableHttp2** |  specify if the server supports HTTP/2 connection.  <br><br>*Optionality*: mandatory <br>Recommendation: Should always be set to `true` |
| **enableRegistry** | control if built-in service registry/discovery is used <br><br>*Optionality*: mandatory <br>Note: Only necessary if running as standalone java -jar xxx |
| **enableParamDescription** | decide if generate parameter description from specifications as annotation. <br><br>*Optionality*: optional Default: `true` |
| **supportDb** | control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml <br><br>*Optionality*: mandatory |
| **dbInfo** | database connection pool configuration info <br><br>*Optionality*: mandatory |
| **supportH2ForTest** | if `true`, add H2 in pom.xml as test scope to support unit test with H2 database. <br><br>*Optionality*: mandatory |
| **supportClient** | if `true`, add com.networknt.client module to pom.xml to support service to service call. <br><br>*Optionality*: mandatory |
| **skipHealthCheck** | decides whether to enable the health check in the handler chain and expose the health check endpoint. Set to `true` to skip the wiring of the health check <br><br>*Optionality*: optional <br>Default: `false` |
| **skipServerInfo** | decides whether to wire the server info in the handler chain and expose the server info endpoint. Set to `true` to skip the wiring of the server info retrieval <br><br><br>*Optionality*: optional <br>Default: `false` |
| **prometheusMetrics** | decides whether to wire the Prometheus metrics collection handler in the handler chain. Set to `true` to skip the wiring of the Prometheus metrics collection handler <br><br>*Optionality*: optional <br>Default: `false` |
| **generateEnvVars** | how environment-based variables should be generated <ul><li>generatedgenerateEnvVars.generate:boolean whether the environment based variables should be generated.</li> Default: `false`.<ul> <li>If set to `false`, config files are copied to target folder (if different from source folder)</li><li>If set to `true`, config values are re-written to environment based</li> </ul><li>valuesgenerateEnvVars.skipArray: boolean whether Arrays in the config files should be re-written or not. This is considered `false` if not set.</li><li>generateEnvVars.skipMap: boolean whether Maps in the config files should be re-written or not. This is considered `false` if not set.</li><li>generateEnvVars.exclude: Array a list of files that should not be re-written </li></ul> Note: This configuration option has been deprecated at 1.6.5 as it is not backward compatible. |
| **specGeneration** | specifies information required for generating openapi specifications from source code.<ul><li>modelPackages: string the codegen tool can only generate specfication for models now. This config item specifies the package names of models in the class path. Mutliple package names are delimited by comma.</li><li>mergeTo: string If there is an existing openapi specification and users wants to merge the generated model sepcifications to it, this config item can be used to specify the location of the existing specification.</li><li>outputFormat: string Specifies the expected output format of the specification. Value can be yaml, json, or both </li><li>outputFilename: string the name of the generated openapi file. If not specified, the output file name is default to openapi_generated.</ul> |


**Here is an example of config.json for openapi generator.**

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
  "generateModelOnly": false,
  "generateValuesYml": false,
  "regenerateCodeOnly":false,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,  
  "enableRegistry": false,
  "enableParamDescription": true,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=`false`",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false,
  "skipHealthCheck": false,
  "skipServerInfo": false,
  "prometheusMetrics": false,
  "generateEnvVars": {
  	"generate": true,
  	"skipArray": true,
  	"skipMap": true,
  	"exclude": [
  		"handerl.yml",
  		"values.yml"
  	]
  },
  "specGeneration": {
  	"modelPackages": "com.networknt.petstore.model",
  	"mergeTo": "/tmp/codegen/openapi.json",
  	"outputFormat": "yaml, json",
  	"outputFilename": "openapi_gen_test"
  }
}
```

In most of the cases, developers will only update handlers, handler tests and models in a generated project. Of course, you might need a different database for your project, and we have a [database tutorial] that can help you to further config Oracle and Postgres.   

Given we have most of our model and config files in [model-config][] repo, most generator input would come from the rest folder in model-config for the light-rest-4j framework.

Let's clone the project to your workspace as we will need it in the following steps. I am using ~/networknt as a workspace, but it can be anywhere in your home directory.


```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```


## Usage

#### Java Command line

Before using the command line to generate the code, you need to check out the light-codegen repo and build it. I am using ~/networknt as the workspace, but it can be anywhere in your home directory.  

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install
```


### JSON Model

Given we have test openapi.json and config.json in light-rest-4j/src/test/resources folder, the following command line will generate a RESTful  API at /tmp/openapi-json folder.

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-json -m light-rest-4j/src/test/resources/openapi.json -c light-rest-4j/src/test/resources/config.json
```

After you run the above command, you can build and start the service:
```
cd /tmp/openapi-json
mvn clean install exec:exec
```

To test the service from another terminal:
```
curl http://localhost:8080/server/info
```

### YAML Model

Given we have test openapi.yaml and config.json in light-rest-4j/src/test/resources folder, the following command line will generate a RESTful  API at /tmp/openapi-yaml folder.

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-yaml -m light-rest-4j/src/test/resources/openapi.yaml -c light-rest-4j/src/test/resources/config.json
```

After you run the above command, you can build and start the service:
```
cd /tmp/openapi-yaml
mvn clean install exec:exec
```

To test the service from another terminal:
```
curl http://localhost:8080/server/info
```


The above example use local OpenAPI specification and config file. Let's try to use files from github.com:

Working directory: light-codegen

```
rm -rf /tmp/openapi-petstore
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-petstore -m https://raw.githubusercontent.com/networknt/model-config/master/rest/openapi/petstore/1.0.0/openapi.json -c https://raw.githubusercontent.com/networknt/model-config/master/rest/openapi/petstore/1.0.0/config.json
```

Please note that you need to use a raw url when accessing github files. The above command line will generate a petstore service in /tmp/openapi-petstore.

Given we have most of the model and config files in model-config repo, most generator input would from the rest folder in model-config. Here is the example to generate petstore. Assuming model-config is in the same workspace as light-codegen.

Working directory: networknt

```
rm -rf /tmp/openapi-petstore
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-petstore -m model-config/rest/openapi/petstore/1.0.0/openapi.json -c model-config/rest/openapi/petstore/1.0.0/config.json

```

* Generate openapi specifications from source code (supports for code-first development)

To support code-first development, we provide the openapi specification generator to generate specifications from code. The command below demonstrates the usage of the specifcation generator.
Please note that the configuration item `specGeneration` is required to generate specifications. For details of this configuration item, please see [Config](#config).

```
cd ~/networknt/light-codegen/code-cli/target
java -cp .:./codegen-cli.jar:/path/to/project/target/classes com.networknt.codegen.Cli -f openapi-spec -o /path/to/config/config.json
```

#### Config Parameterization
To support generating parameterized config file, we provide additional commend `-p`, `--parameterize` for codegen-cli
```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -p examplePath/paramsCondig -f openapi -o /tmp/openapi-json -m light-rest-4j/src/test/resources/openapi.json -c light-rest-4j/src/test/resources/config.json
```
By executing the above command, a project containing custom parameterized configuration files will be generated. Each configuration file that exists on the specified path `examplePath/paramsCondig` will be parameterized and added to the target configuration file path.

For more details of config parameterization, please see [Config](#config).

#### Docker Command Line

Above local build and command line utility works but it is very hard to use that in devops script. In order to make scripting easier, we have dockerized the command line utility.


The following command is using docker image to generate the code into /tmp/light-codegen/generated:
```
docker run -it -v ~/networknt/light-codegen/light-rest-4j/src/test/resources:/light-api/input -v /tmp/light-codegen:/light-api/out networknt/light-codegen -f openapi -m /light-api/input/openapi.json -c /light-api/input/config.json -o /light-api/out/generated
```
On Linux environment, the generated code might belong to root:root and you need to change the owner to yourself before building it.

```
cd /tmp/light-codegen
sudo chown -R steve:steve generated
cd generated
mvn clean install exec:exec
```
To test it.
```
curl localhost:8080/v1/pets/111
```

#### Docker Scripting

You can use docker run command to call the generator but it is very complicated for the parameters. In order to make things easier and friendlier to DevOps flow let's create a script to call the command line from docker image.

If you look at the docker run command you can see that we basically need one input folder for schema and config files and one output folder to generated code. Once these volumes are mapped to local directory and with framework specified, it is easy to derive other files based on convention.


```
cd model-config
./generate.sh openapi ~/networknt/model-config/rest/openapi/petstore/1.0.0 /tmp/petstore
```
Now you should have a project generated in /tmp/petstore/genereted

#### Codegen Site

The service API is ready. We are working on the UI with a generation wizard.


[openapi-parser]: https://github.com/networknt/openapi-parser
[Swagger 2.0 generator tutorial]: /tutorial/generator/swagger/
[Swagger Editor]: /tool/swagger-editor/
[Swagger CLI]: /tool/swagger-cli/
[database tutorial]: /tutorial/rest/swagger/database/
[model-config]: https://github.com/networknt/model-config
