---
title: "Native Image"
date: 2021-01-26T13:16:36-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The number one complaint users have when building AWS Lambdas with JVM languages is the cold start latency issue. 

### Fatjar

GraalVM needs to run against a fat jar file with the main entry point provided by the [custom-runtime][]. 

In the build.gradle, the following lines are important to generate the fat jar. 

```
plugins {
    ...
    id "com.github.johnrengelman.shadow" version "5.2.0"
}

dependencies {
	...	
    implementation 'com.networknt:custom-runtime:2.0.23-SNAPSHOT'
    ...
}

jar {
  manifest {
    attributes 'Main-Class': 'com.networknt.aws.lambda.Runtime'
  }
}
```

### GraalVM Build

GraalVM works by taking the fat jar file and creating a Linux executable file that AWS Lambda can run. 

##### reflect.json

GraalVM does not support refection, so we need to provide a reflect.json to specify all the initialized classes based on the Java Reflection. 

Here is an example. 

```

[
  {
    "name": "com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent",
    "methods": [
      { "name": "<init>", "parameterTypes": [] }
    ],
    "allPublicMethods" : true
  },
  {
    "name": "com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent$ProxyRequestContext",
    "methods": [
      { "name": "<init>", "parameterTypes": [] }
    ],
    "allPublicMethods" : true
  },
  {
    "name": "com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent$RequestIdentity",
    "methods": [
      { "name": "<init>", "parameterTypes": [] }
    ],
    "allPublicMethods" : true
  },
  {
    "name": "com.networknt.aws.lambda.AuthPolicy",
    "allPublicMethods" : true
  },
  {
    "name": "com.networknt.aws.lambda.DefaultResponse",
    "allPublicMethods" : true
  },
  {
    "name": "com.networknt.aws.lambda.InvocationResponse",
    "allPublicMethods" : true
  },
  {
    "name": "com.networknt.aws.lambda.LambdaContext",
    "allPublicMethods" : true
  },
  {
    "name": "com.networknt.petstore.handler.App",
    "allDeclaredConstructors": true,
    "allPublicConstructors": true,
    "allDeclaredMethods": true,
    "allPublicMethods": true
  },
  {
    "name": "ch.qos.logback.core.ConsoleAppender",
    "allPublicMethods":true,
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.core.FileAppender",
    "allPublicMethods":true,
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.encoder.PatternLayoutEncoder",
    "allPublicMethods":true,
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LineOfCallerConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.DateConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.ExtendedThrowableProxyConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LevelConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LineSeparatorConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LoggerConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.MessageConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.ThreadConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.MDCConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.ClassOfCallerConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  },
  {
    "name": "ch.qos.logback.classic.pattern.MethodOfCallerConverter",
    "methods":[{"name":"<init>","parameterTypes":[] }]
  }
]

```

##### reource-config.json

We also need to specify all the resources used by the lambda function. 

resource-config.json
```
{
  "resources": {
    "includes": [
      {"pattern": "app.yml"},
      {"pattern": "logback.xml"}
    ],
    "excludes": [
    ]
  }
}
```

##### bootstrap

A bootstrap file is required to invoke the binary native image as AWS assumes this file is the entry point. 

bootstrap
```
#!/bin/sh
set -euo pipefail
./server

```

##### Makefile

When using custom runtime for the lambda function, the `sam build` will try to locate a Makefile in the lambda function folder to start the build. 

Here is the example in the template.yaml with `Runtime: provided`

```
  PetsGetFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      
      CodeUri: PetsGetFunction
      Handler: com.networknt.petstore.handler.App::handleRequest
      Runtime: provided
      MemorySize: 512
      FunctionName: PetsGetFunction
      Environment: # More info about Env Vars: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#environment-object
        Variables:
          PARAM1: VALUE
      
      Events:
        PetsGet:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            Path: /v1/pets
            Method: GET
            
            RestApiId:
              Ref: ApiGateway

```

Here is the Makefile

```

CUR_DIR := $(abspath $(patsubst %/,%,$(dir $(abspath $(lastword $(MAKEFILE_LIST))))))

build-PetsGetFunction:
	cd $(CUR_DIR) && ./gradlew build
	cp $(CUR_DIR)/build/graalvm/server $(ARTIFACTS_DIR)
	cp $(CUR_DIR)/bootstrap $(ARTIFACTS_DIR)
```

##### build.gradle

The above Makefile calls the `gradlew build` to build the native image. The build.grale has a section. 

```
task buildGraalvmImage {
 inputs.files("${project.projectDir}/src/main", configurations.compileClasspath)
 outputs.upToDateWhen {file("${buildDir}/graalvm/server").exists()}
 outputs.file file("${buildDir}/graalvm/server")

 doLast {
    exec {
      commandLine "bash", "-c", "chmod +x build_graalvm.sh; chmod +x bootstrap; ./build_graalvm.sh"
    }
  }
}

buildGraalvmImage.dependsOn shadowJar, test
build.dependsOn buildGraalvmImage

```

##### build_graalvm.sh

The build.gradle calls the build_graalvm.sh to build the native image with docker.

```

#!/bin/bash

docker run --rm --name graal -v $(pwd):/working springci/graalvm-ce:master-java11 \
    /bin/bash -c "native-image \
                    --enable-url-protocols=http,https \
                    --no-fallback \
                    --allow-incomplete-classpath \
                    --enable-all-security-services \
                    -H:ReflectionConfigurationFiles=/working/reflect.json \
                    -H:ResourceConfigurationFiles=/working/resource-config.json \
                    -H:+ReportExceptionStackTraces \
                    -jar /working/build/libs/PetsGetFunction-all.jar \
                    ; \
                    cp PetsGetFunction-all /working/build/graalvm/server"

mkdir -p build/graalvm
if [ ! -f "build/graalvm/server" ]; then
    echo "there was an error building graalvm image"
    exit 1
fi

```

With the above config files and build scripts, we can invoke the build from the parent fold with AWS CLI to build multiple Lambda functions. 

Run the following command in the folder with template.yanl file. 
```
sam build
```

[custom-runtime]: /style/light-aws-lambda/custom-runtime/
