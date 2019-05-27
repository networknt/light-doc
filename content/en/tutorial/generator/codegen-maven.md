---
title: "Code generation with maven build process"
date: 2018-03-11T19:48:21-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Besides using java command line to generate microservice API based on the OpenAPI 3.0 specification, light-codegen also can be integrated with the project maven build process.

Sometimes the specification could be changed by different team members frequently. In this case, it could be challenging to re-generate the API project from the Java command line for every specification change.

To serve this purpose a new way is introduced here which could integrate the light-codegen into the maven build process. You can find the reference example at [petstore maven codegen example][].


#### Work Steps

- Initiate the microservice API project. Define one module for the service and another module for specification

- create or copy the specification and the config JSON file to the specification module, for example, /petstore-spec/config

- In the specification module, add maven plugin to run the light-codegen [pom example in the petstore project][]

```xml
             <plugin>
                 <groupId>org.codehaus.mojo</groupId>
                 <artifactId>exec-maven-plugin</artifactId>
                 <version>1.4.0</version>
                 <executions>
                     <execution>
                         <id>light4j-codegen</id>
                         <phase>process-resources</phase>
                         <goals>
                             <goal>exec</goal>
                         </goals>
                         <inherited>false</inherited>
                         <configuration>
                             <executable>java</executable>
                             <arguments>
                                 <argument>-jar</argument>
                                 <argument>${settings.localRepository}/com/networknt/codegen-cli/${version.light-4j}/codegen-cli-${version.light-4j}.jar</argument>
                                 <argument>-f</argument>
                                 <argument>openapi</argument>
                                 <argument>-o</argument>
                                 <argument>../petstore-service</argument>
                                 <argument>-m</argument>
                                 <argument>config/openapi.yaml</argument>
                                 <argument>-c</argument>
                                 <argument>config/config.json</argument>
                             </arguments>
                         </configuration>
                     </execution>
                 </executions>
             </plugin>
```

- At the first time to run the build process to generate an initial service module, set config value "specChangeCodeReGenOnly" as false from config JSON file:

  ```json
  {
    "name": "petstore",
    "version": "3.0.1",
    "groupId": "com.networknt",
    "artifactId": "petstore",
    "rootPackage": "com.networknt.petstore",
    "handlerPackage":"com.networknt.petstore.handler",
    "modelPackage":"com.networknt.petstore.model",
    "overwriteHandler": true,
    "overwriteHandlerTest": true,
    "overwriteModel": true,
    "httpPort": 8080,
    "enableHttp": false,
    "httpsPort": 8443,
    "enableHttps": true,
    "enableHttp2": true,
    "enableRegistry": false,
    "supportDb": false,
    "supportH2ForTest": false,
    "supportClient": false,
    "specChangeCodeReGenOnly": false,
    "dockerOrganization": "networknt"
  }
  ```

- Run maven install to build the project, and the maven build will trigger the light-codegen to generate the initial service into the service module

   ```
      cd ~/networknt/light-example-4j/rest/perstore-with-codegen

      mvn clean install

   ```

- After the service project generated, developers start to work on the service; then change  config value "specChangeCodeReGenOnly" to true:

  ```
    "specChangeCodeReGenOnly": true,

  ```

- The light-codegen will be triggered every time maven install be executed; but since the "specChangeCodeReGenOnly" set as true, only specification change related code will be generated to the service project.

  For example, data model, endpoint, handler.yml...


[petstore maven codegen example]: https://github.com/networknt/light-example-4j/tree/develop/rest/perstore-with-codegen
[pom example in the petstore project]: https://github.com/networknt/light-example-4j/blob/develop/rest/perstore-with-codegen/petstore-spec/pom.xml
