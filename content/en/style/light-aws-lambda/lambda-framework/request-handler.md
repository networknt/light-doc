---
title: "Request Handler"
date: 2021-02-07T16:35:48-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This is the entry point for the Lambda function when the lambda framework is used. It intercepts the request and applies request side cross-cutting concerns, calls the business handler to perform the Lambda function's business logic, and then intercepts the response with response side cross-cutting concerns. 

We call these cross-cutting concerns middleware handlers and here is the list in calling sequence.

### scope-verifier

The [scope-verifier][] is responsible for verifier the scopes in the JWT tokens against the scopes defined in the OpenAPI specification endpoint. This middleware handler is the first to be called.


### lambda-validator

Once the scope is verifier, then we need to invoke the [lambda-validator][] to validate the request against the OpenAPI specification with [json-schema-validator][]. 



[scope-verifier]: /style/light-aws-lambda/lambda-framework/scope-verifier/
[lambda-validator]: /style/light-aws-lambda/lambda-framework/lambda-validator/
[json-schema-validator]: https://github.com/networknt/json-schema-validator
