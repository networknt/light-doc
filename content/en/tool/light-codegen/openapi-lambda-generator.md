---
title: "OpenAPI Lambda Generator"
date: 2021-01-19T14:33:05-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This generator is based on the OpenAPI 3.0 specification to generate one AWS Lamdbd function per endpoint. Also, it generates all the artifacts for the building and deploying with cloud formation YAML files.

## Input

#### Model

In light-rest-4j framework generator, the model that drives code generation is the OpenAPI 3.0 specification previously named Swagger specification. When editing it, it usually will be in YAML format with separate files for readability and flexibility. Before leverage it in the light-rest-4j framework, all YAML files need to be bundled to a single file in YAML or JSON format to be consumed by the framework and generator. Also, validation needs to be done to make sure that the openapi.yaml or openapi.json is valid against JSON schema of the OpenAPI 3.0 specification.

Note: currently, we support only the OpenAPI 3.0 specification and Swagger 2.0 specification support was removed. 

- [Swagger Editor][]

- [Swagger CLI][]


#### Config

<!-- Reference Table -->

| Field Name | Description |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **projectName** | Used in generated prom.xml for project name <br><br> *Optionality*: mandatory |
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
| **useLightProxy** | replace the AWS API Gateway with light-proxy for cross-cutting concerns if it is `true`<br><br>*Optionality*: optional Default: `false` |
| **launchType** | specify if `EC2` or `Fargate` will be used to launch the light-proxy container. <br><br>*Optionality*: mandatory |
| **publicVpc** | use public subnet for the VPC to deploy the light-proxy if it is `true`. <br><br>*Optionality*: optional Default: `true` |
| **packageDocker** | package the Lambda function with Docker image instead of zip file for deployment if it `true`. <br><br>*Optionality*: mandatory <br>Recommendation: `false`|
| **region** |  specify the AWS region for the Lambda fuctions and/or light-proxy deployment.  <br><br>*Optionality*: mandatory <br>Example: `us-east-2` |
| **enableRegistry** | control if built-in service registry/discovery is used <br><br>*Optionality*: mandatory <br>Note: Only necessary if running as standalone java -jar xxx |
| **prometheusMetrics** | decides whether to wire the Prometheus metrics collection handler in the handler chain. Set to `true` to skip the wiring of the Prometheus metrics collection handler <br><br>*Optionality*: optional <br>Default: `false` |


**Here is an example of config.json for openapi generator.**

```json
{
  "projectName": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "useLightProxy": true,
  "launchType": "EC2",
  "publicVpc": true,
  "packageDocker": false,
  "region": "us-east-2",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "enableRegistry": false,
  "generateModelOnly": false
}
```

Given we have most of our model and config files in [model-config][] repo, most generator input would come from the lambda folder in model-config for the light-aws-lambda framework.

Let's clone the project to your workspace as we will need it in the following steps. I am using `~/networknt` as a workspace, but it can be anywhere in your home directory.


```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```

## Usage

#### Java Command line

You can [download or build][] the codegen-cli command line jar. 

* JSON Model

Given we have test openapi.json and config.json in light-rest-4j/src/test/resources folder, the following command line will generate a RESTful  API at /tmp/openapi-lambda folder.

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapilambda -o /tmp/openapi-lambda -m light-rest-4j/src/test/resources/openapi.json -c light-rest-4j/src/test/resources/config.json
```

After you run the above command, you can build and start the service:
```
cd /tmp/openapi-lambda
```

To test the service from another terminal:
```
curl http://localhost:8080/server/info
```

* YAML Model

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
java -jar codegen-cli/target/codegen-cli.jar -p examplePath/paramsCondig -f openapilambda -o /tmp/openapi-json -m light-rest-4j/src/test/resources/openapi.json -c light-rest-4j/src/test/resources/config.json
```
By executing the above command, a project containing custom parameterized configuration files will be generated. Each configuration file that exists on the specified path `examplePath/paramsCondig` will be parameterized and added to the target configuration file path.

For more details of config parameterization, please see [Config](#config).

#### Codegen Site

You can generate a Lambda project from the site https://codegen.lightapi.net with your OpenAPI specification and a config file. 

[openapi-parser]: https://github.com/networknt/openapi-parser
[Swagger Editor]: /tool/swagger-editor/
[Swagger CLI]: /tool/swagger-cli/
[database tutorial]: /tutorial/rest/swagger/database/
[model-config]: https://github.com/networknt/model-config
[download or build]: /tool/light-codegen/download-build/
