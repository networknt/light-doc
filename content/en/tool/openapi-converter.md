---
title: "OpenAPI Converter"
date: 2018-03-12T09:44:28-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

OpenAPI 3.0 specification was released in late 2017, and it resolved several major issues or limitations of Swagger 2.0 specification. The community is cheered about the release; however, it is very hard for developers to adopt it at the moment, even after several months since the release. Given the new specification is not backward compatible, all tools around Swagger 2.0 need to be rewritten, and it takes a lot of time. 

In our example, we have added OpenAPI 3.0 support in [light-rest-4j][] framework in late 2017 and we have to build our own [openapi-parser][] to do the runtime validation again the spec. 

Over the last couple of weeks, a lot of customers asked questions on how to migrate from the existing Swagger 2.0 API to OpenAPI 3.0 API. Also, our team has to upgrade existing tutorials and examples to OpenAPI 3.0 in light-example-4j/rest/openapi/

This document is the first step in the process that converts the existing Swagger 2.0 specification to OpenAPI 3.0 specification.

The tool we are using is an online service called [openapi-coverter][]

Although it has a lot of extra features, only the convert function is explored here. 

The process is simple: 

* Copy/paste the Swagger 2.0 yaml file into the text area.
* Click "Convert Swagger/OpenAPI 2.0"
* The converted specification will be displayed as text.
* Copy/paste the converted result into your swagger-editor for post-processing.
* Don't forget to update the examples to leverage [light-codegen response from example][].
* Generate a new project from the OpenAPI 3.0 specification.
* Move all the handlers and handler tests from old projects
* Enjoy OpenAPI 3.0

We have followed above process to upgrade [ms-aggregate tutorial][] from Swagger 2.0 to OpenAPI 3.0 specification.

[light-rest-4j]: /style/light-rest-4j/
[openapi-parser]: /tool/openapi-parser/
[openapi-coverter]: https://mermade.org.uk/openapi-converter
[light-codegen response from example]: /tutorial/generator/openapi-mock/
[ms-aggregate tutorial]: /tutorial/rest/openapi/ms-aggregate/
