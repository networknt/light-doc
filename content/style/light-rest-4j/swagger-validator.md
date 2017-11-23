---
title: "Swagger Validator"
date: 2017-11-22T22:45:59-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This handler is part of the [light-rest-4j][] which is built on top of light-4j 
but focused on RESTful API only. Also, it only works with Swagger 2.0 specification.

It encourages design driven implementation so swagger specification should be 
done before the implementation starts. With the [light-codegen][] 
light-4j generator, the server stub can be generated and start running within seconds. 
However, we cannot rely on generator for validation as specification will be 
changed along the life cycle of the API. This is why we have provided a validator 
that works on top of the specification at runtime. In this way, the generator 
should only be used once and the validator will take the latest specification and 
validate according the specification at runtime. 

### Fail fast

As you may noticed that our Status object only supports one code and message. 
This is the indication the framework validation is designed as fail fast. 
Whenever there is an error, the server will stop processing the request and 
return the error to the consumer immediately. There are two reasons on this 
design and it is documented in [fail-fast][] in design section. 


### ValidatorHandler

This is the entry point of the validator and it is injected during server 
start up if validator.yml enableValidator is true. By default, only 
RequestValidator will be called. Response validation should be done in the
[client][] module. 

### RequestValidator

It will validate the following:

* uri
* method
* header
* query parameters
* path parameters
* body if available

When necessary, [json-schema-validator][] will be called to do json schema validation.

# ResponseValidator

It will validate the following:

* header
* response code
* body if available

when necessary, [json-schema-validator][] will be called.

### SchemaValidator

If schema is defined in swagger.json, then the [json-schema-validator][] 
will be called to validate the input against a json schema defined in draft v4.

### Test

In order to test validator, the test suite starts a light-4j server and serves 
petstore api for testing. It is a demo on how to unit test your API during 
development.


[light-rest-4j]: https://github.com/networknt/light-rest-4j
[light-codegen]: /tool/light-codegen/
[fail-fast]: /architecture/fail-fast/
[client]: /concern/client/
[json-schema-validator]: https://github.com/networknt/json-schema-validator
