---
title: "Rest Examples"
date: 2017-11-22T21:59:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

As light-rest-4j is the first framework supported, we have more Restful examples than any other style of microservices. All rest examples can be found at [light-example-4j][]. Under the rest folder there are two folders to differentiate Swagger 2.0 and OpenAPI 3.0 implementations. 

If you are interested in implementing your Restful services in Kotlin, you can find examples in [light-example-kotlin][] repository openapi folder. 

As we are encouraging the [design-driven][] approach to build services, you can find the corresponding specifications for these examples as well as light-codegen config files in the [model-config][] repository.  There is also a README.md with general information about each service and command lines to generate Java or Kotlin service for [light-codegen][]. 

Most example applications consist of multiple services, and only the petstore is a single service which is described in [getting started][]. Other examples are used in [tutorial][], and you can follow them step by step into the details.


[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/rest
[getting started]: /getting-started/light-rest-4j/
[tutorial]: /tutorial/rest/
[design-driven]: /design/design-first/
[light-example-kotlin]: https://github.com/networknt/light-example-kotlin/tree/master/openapi
[model-config]: https://github.com/networknt/model-config
[light-codegen]: https://github.com/networknt/light-codegen
