---
title: "OpenAPI Display"
date: 2018-10-22T22:46:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the framework, [light-rest-4j][] providers two handlers to display OpenAPI Specification for the service API.
It only works with OpenAPI 3.0 specification.


### OpenApi display Handler

1. SpecDisplayHandler - Display specification content directly from browser.

2. SpecSwaggerUIHandler -Display specification with Swagger UI.



### Config

A specification file openapi.json or openapi.yaml file should be in the config folder of your API
implementation and it will be loaded to memory with OpenApiHelper during server 
start up.


The specification display handlers have a config yml file (specification.yml) which define the specification name and content type:



```
---
# Specification type and file name definition

fileName: config/openapi.yaml
contentType: text/yaml

```

If the service API has different name and content type for the specification, use can extend the config yml file (specification.yml) to define service's specification file name and content type


### Path for Specification display

With the [light-codegen][] of light-4j generator, the generated handler.yml will include the handlers and paths for the specification display


Handlers:

```
  # Framework endpoint handlers

  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui

```

Paths:


```
  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec

  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```


[light-rest-4j]: https://github.com/networknt/light-rest-4j
[light-codegen]: /tool/light-codegen/