---
title: "Codegen Proxy"
date: 2021-01-26T10:52:16-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For more info about the lambda generator, please visit the [openapi-lambda-generator][]. 

When using the light-codegen to generate a lambda project with light-proxy, you can use the following config.json as a template. 

```
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

The above [config.json](https://github.com/networknt/model-config/blob/master/lambda/petstore/config-proxy.json) is in the model-config repo with a [README.md](https://github.com/networknt/model-config/tree/master/lambda/petstore) for codegen cli. 

One of the [generated project][] can be found at light-example-4j/lambda/petstore-proxy folder. 


[openapi-lambda-generator]: /tool/light-codegen/openapi-lambda-generator/
[generated project]: https://github.com/networknt/light-example-4j/tree/master/lambda/petstore-proxy
