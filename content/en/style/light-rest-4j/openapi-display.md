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
reviewed: true
---

In the [light-rest-4j][] framework, we have provided two handlers to display OpenAPI Specification for the service API. It only works with OpenAPI 3.0 specifications.


### OpenApi display Handler

1. SpecDisplayHandler - Load specification content directly from the browser.

As the OpenAPI 3.0 specification can be named as `openapi.yaml`, `openapi.yml` or `openapi.json`,  And it can be located anywhere in different deployment environments. We have this handler to load the specification based on the specification.yml configuration. It allows the SpecSwaggerUIHandler to refer to the specification with a static endpoint defined in the handler.yml file as `spec.yaml`. 


2. SpecSwaggerUIHandler -Display specification with Swagger UI on the browser.

This handler output the SwaggerUI with HTML and javascript to render the Swagger UI based on the specification loaded from the above handler. The site HTML is hard-coded in the handler with reference to the '/spec.yaml' defined in the Paths section below. 

### Config

A specification file `openapi.json` or `openapi.yaml` file should be in the config folder of your API implementation, and it will be loaded to memory with OpenApiHelper during server startup.


The specification display handlers have a config yml file (specification.yml) which define the specification name and content-type:


```
---
# Specification type and file name definition

fileName: config/openapi.yaml
contentType: text/yaml

```

Suppose the service API has a different name and content type for the specification. In that case, users can customize the config file (specification.yml) to define the service's specification file name and content type.


### Path for Specification display

With the [light-codegen][] of the light-4j generator, the generated handler.yml will include the handlers and paths for the specification display.


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

The above paths are part of the Swagger UI rendering, and you can enable security or other cross-cutting concerns if you want. These handlers will be generated for all the light-rest-4j projects if you are using the light-codegen. They can also be added to the light-proxy and light-router to serve the Swagger UI for the services behind them. 

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[light-codegen]: /tool/light-codegen/
