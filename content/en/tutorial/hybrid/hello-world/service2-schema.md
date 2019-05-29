---
title: "Service2 Schema"
date: 2019-04-16T09:17:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have completed the service1 and [deployed it][] with the server instance. In this step, we are going to create another service and deploy it along with service1. 

To create service2, we need to create a schema.json and a config.json first. The final version can be found at https://github.com/networknt/model-config/tree/master/hybrid/hello-world/service2

The config.json is almost the same as service1. The only difference is the name of the service and artifactId. 

```json
{
  "rootPackage": "com.networknt.hello",
  "handlerPackage":"com.networknt.hello.handler",
  "modelPackage":"com.networknt.hello.model",
  "artifactId": "service2",
  "groupId": "com.networknt",
  "name": "service2",
  "version": "1.0.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": false,
  "supportPostgresql": false,
  "supportH2ForTest": false,
  "supportClient": false
}
```

The following is the schema.json for the service2.

```json
{
  "host": "lightapi.net",
  "service": "service2",
  "action": [
    {
      "name": "command",
      "version": "0.1.0",
      "handler": "Command",
      "scope" : "command.w",
      "schema": {
        "title": "Service2",
        "type": "object",
        "properties": {
          "source": {
            "type": "string"
          },
          "target": {
            "type": "string"
          }
        },
        "required": ["source", "target"]
      }
    },
    {
      "name": "command",
      "version": "0.1.1",
      "handler": "Command",
      "scope" : "command.w",
      "schema": {
        "title": "Service2",
        "type": "object",
        "properties": {
          "source": {
            "type": "string"
          },
          "target": {
            "type": "string"
          }
        },
        "required": ["source", "target"]
      }
    }
  ]
}
```

The above schema has two actions. The action names are the same, but only versions are different. It is to demo how you are going to add a new action if it is not backward compatible with the original one. 

With both the config.json and schema.json, let's [generate service2][] with the light-codegen in the next step. 


[deployed it]: /tutorial/hybrid/hello-world/deploy-service1/
[generate service2]: /tutorial/hybrid/hello-world/generate-service2/
