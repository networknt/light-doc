---
title: "Schema Validation"
date: 2021-10-19T14:57:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Several components are related to the REST API request/response validation, and they all serve different purposes.

### json-schema-validator

json-schema-validator is a green library that is responsible for validating the JSON against the JSON schema. We are using it to validate the incoming request (headers, query parameters, path parameters and body) extracted from the OpenAPI specification. I am guessing this is the validator you are looking for. It can flatten the $ref, but it is during the runtime. When this is running on a server, you might not have access to the Internet to resolve the remote references. That is why we recommend you have the $ref resolved locally in one specification to avoid remote access. This has been done in the next module called openapi-parser.


https://github.com/networknt/json-schema-validator


### openapi-parser

openapi-parse is a library to parse the OpenAPI 3.0 specification and prepare it with an internal structure to assist validation and security of the light-4j server. This library can resolve the $ref in the same specification—for example, all the components definitions. If your specification has several files or even remote references, you should resolve it before using it to generate a light-4j project. To assist enterprise users who are managing specifications with several files that share common definitions, we have developed openapi-bundler.



https://github.com/networknt/openapi-parser


### openapi-bundler

openapi-bundler is a command-line tool that combines several specification files into one and resolves all remote references to local references to create a final specification as the input for the light-4j codegen.


https://github.com/networknt/openapi-bundler

