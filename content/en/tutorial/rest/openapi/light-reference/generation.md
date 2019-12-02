---
title: "Light Reference Specification and Code Generation"
date: 2019-12-01T23:04:23-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before building the reference API, we need to define the specification, which is the contract with the consumer. This is a RESTful API, and we are going to use the swagger-editor to create it. 

The final specification can be found at https://github.com/networknt/model-config/tree/master/rest/light-reference

Along with openapi.yaml, there is a config.json to control how to generate the code from light-codegen. In the README.md, there is a command-line to show users how to use the locally built light-codegen to generate the light-reference project. 

You can also use the online https://codegen.lightapi.net to generate the project and unzip it into the light-reference folder. 

For more information to use light-codegen, please refer to the [codegen tutorial](tutorial/generator/).

With the project generated, we are going to [mock][] one of the handlers to output the JSON response RcSelect requires. 

[mock]: /tutorial/rest/openapi/light-reference/mock-handler/

