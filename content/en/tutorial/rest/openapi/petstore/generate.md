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

We have built [light-codegen][] to support project scaffolding for all the frameworks 
provided by the light platform. We believe that in order to build scalable enterprise 
applications and microservices, you have to adopt a [design first][] methodology. 
There are numerous ways to use light-codegen but for the purposes of this tutorial, 
we are using it as a command line tool.
 
This project will be updated periodically when a new version of the light-4j framework is 
released or if any updates are made to [light-codegen][]. Both [swagger petstore][] and 
[openapi petstore][] projects are our test beds for [light-rest-4j][].

Here is the command to generate this project from the model-config directory. It assumes that 
you have light-example-4j cloned to the same working directory and that the petstore 
directory is moved or renamed. If you have changed the specification and want to regenerate 
the project, it is better to generate it into another folder and then do a full-text 
comparison to merge the new project into the old project.  

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/petstore -m model-config/rest/openapi/petstore/1.0.0/openapi.json -c model-config/rest/openapi/petstore/1.0.0/config.json
```

The cli tool needs to specify:
 
* -f openapi       - choose rest framework that support OpenAPI 3.0 specification for the generator
* -o output        - choose the output folder for the generated project
* -m model         - choose the model definition file or IDL spec as it is design driven generator
* -c config        - choose config file to control how the project is generated


For information on how to use light-codegen, please check out the [getting started][] page. The
detailed [documentation][] can be found in the tools section.

With the project generated, our next step is to [build and start][] the server. 

[design first]: /design/design-first/
[light-codegen]: https://github.com/networknt/light-codegen
[light-rest-4j]: /style/light-rest-4j/
[getting started]: /getting-started/light-codegen/
[documentation]: /tool/light-codegen/
[build and start]: /tutorial/rest/openapi/petstore/build/
[swagger petstore]: https://github.com/networknt/light-example-4j/tree/master/rest/swagger/petstore
[openapi petstore]: https://github.com/networknt/light-example-4j/tree/master/rest/openapi/petstore

