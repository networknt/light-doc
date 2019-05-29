---
title: "Service1 Schema"
date: 2019-04-16T13:10:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [generated a server][]. In this section, we are going to create a schema for service1 so that we can use it to scaffold service1 project. 

Let's call this service as `query`, and there are two actions for the service. An action is just like an endpoint in Swagger specification. 

You can find the schema.json for service1 at https://github.com/networknt/model-config/tree/master/hybrid/hello-world/service1

Here is the schema.json for the service1. 

```json
{
  "host": "lightapi.net",
  "service": "service1",
  "action": [
    {
      "name": "query",
      "version": "0.1.0",
      "handler": "Query",
      "scope" : "query.r",
      "schema": {
        "title": "Service1",
        "type": "object",
        "properties": {
          "param1": {
            "type": "string"
          },
          "param2": {
            "type": "string"
          }
        },
        "required": ["param1", "param2"]
      }
    },
    {
      "name": "info",
      "version": "0.1.0",
      "handler": "Info",
      "scope" : "info.r",
      "schema": {
        "title": "Service1",
        "type": "object",
        "properties": {
          "filter": {
            "type": "string"
          }
        },
        "required": ["filter"]
      }
    }
  ]
}

```

The following properties are defined in the schema.json. 

* host

For each service, you have a host which is the domain for the service. You can have two different services that belong to two different domains deployed to one server instance. 

* service

Each service will have a unique identifier. For one schema file, the host and service are the same and shared by all actions/endpoints. 

* action

Each service can have multiple actions defined as array items. Each action has a name and a version which uniquely identify the action/endpoint. You can have two actions with the same name but different versions if the newer version is not backward compatible. 

The handler in action will define the class name of the handler for the action in the generated project. 

The scope is a concept of OAuth 2.0 specification to control which client can access the action. 

The schema is a JSON schema which defines the body of the request. The light-hybrid-4j will use it to validate the request. 

For this particular example, we have two actions: query and info.

With the schema.json is ready for service1, let's create a config.json and [generate the service1][] in the next step.

[generated a server]: /tutorial/hybrid/hello-world/generate-server/
[generate the service1]: /tutorial/hybrid/hello-world/generate-service1/
