---
title: "Parameter Serialization"
date: 2019-05-13T15:16:43-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For most users, their use cases are straightforward in term of parameters; however, we do have some users have advanced use cases with [parameter serialization][]. 

OpenAPI 3.0 supports arrays and objects in operation parameters (path, query, header, and cookie) and lets you specify how these parameters should be serialized. The serialization method is defined by the `style` and `explode` keywords:

* `style` defines how multiple values are delimited. Possible styles depend on the parameter location – path, query, header or cookie.

* `explode` (true/false) specifies whether arrays and objects should generate separate parameters for each array item or object property.

OpenAPI serialization rules are based on a subset of URI template patterns defined by RFC 6570. Tool implementers can use existing URI template libraries to handle the serialization, as explained below.

In this tutorial, we are going to include some examples so that users can learn from them. 

* [JSON query parameter](/tutorial/rest/openapi/parameter-serialization/json-query-param/)

[parameter serialization]: https://swagger.io/docs/specification/serialization/

