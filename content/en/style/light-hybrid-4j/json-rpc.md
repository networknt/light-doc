---
title: "JSON RPC"
date: 2018-03-04T11:42:10-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

You can use JSON to define the schema for each service and the server will use it to validate
the request body. 

There are two different parts in the request body: metadata and data. The schema only applies to the data portion.

Here is an example of schema. 

```json
{
  "host": "lightapi.net",
  "service": "certification",
  "action": [
    {
      "name": "certifyEndpoint",
      "version": "0.1.0",
      "handler": "Endpoint",
      "scope" : "cert.endpoint.r",
      "schema": {
        "title": "Endpoint Certification",
        "type": "object",
        "properties": {
          "host": {
            "type": "string"
          },
          "path": {
            "type": "string"
          },
          "token": {
            "type": "string"
          },
          "environment": {
            "type": "string", 
            "enum": ["test", "prod"]
          }
        },
        "required": ["host", "path"]
      }
    },
    {
      "name": "certifyRegistry",
      "version": "0.1.0",
      "handler": "Registry",
      "scope" : "cert.registry.r",
      "schema": {
        "title": "Registry Certification",
        "type": "object",
        "properties": {
          "registryUrl": {
            "type": "string"
          },
          "environment": {
            "type": "string",
            "enum": ["test", "prod"]
          }
        },
        "required": ["registryUrl"]
      }
    }
  ]
}

``` 

### How schema is loaded

Each service has a scheme file in the resources folder in the jar file. Multiple jar files
can be put into a service folder that is part of the classpath of a hybrid server. When server
is started, it will iterate all jar files in the classpath to load schema.json and mergge them
together in memory. 

Given the above loading logic, you cannot put the service jar file into a fat jar of the server
as schema.json will be overwritten and only one is available at the end. 

### Json schema validation

Schema validation is done with [json-schema-validator][].
 
 
[json-schema-validator]: https://github.com/networknt/json-schema-validator
 