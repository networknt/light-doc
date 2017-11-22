---
title: "Generate"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 40
aliases: []
toc: false
draft: false
---

We have [light-codegen][] to support project scaffolding for all the frameworks provided
by light platform. There are numeric ways to use light-codegen and here we are just
using it as command line cli tool.  


This project will be updated constantly when a new version of light-4j framework 
is released or any updates in [light-codegen][].

Here is the command line to generate this project from model-config directory. It
assumes that you have light-example-4j cloned in the same working directory and 
petstore directory is removed or renamed. If you have changed the specification
and want to regenerate the project, it is better to generate into another folder
and then do a full text comparison to merge the old project and new project. 

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/petstore -m model-config/rest/openapi/petstore/1.0.0/openapi.json -c model-config/rest/openapi/petstore/1.0.0/config.json
```

The cli tool needs to specify:
 
* -f openapi       - choose rest framework that support OpenAPI 3.0 specification for the generator
* -o output        - choose the output folder for the generated project
* -m model         - choose the model definition file or IDL spec as it is design driven generator
* -c config        - choose config file to control how the project is generated


For information on how to use light-codegen, please check out the [getting started][]. The
detailed [document][] can be found at tools section. 

[light-codegen]: https://github.com/networknt/light-codegen
[getting started]: /getting-started/light-codegen/
[document]: /tool/light-codegen/