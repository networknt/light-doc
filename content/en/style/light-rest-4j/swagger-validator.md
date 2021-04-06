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
reviewed: true
---


This handler is part of the [light-rest-4j][] which is built on top of light-4j but focused on RESTful API only. Also, it only works with Swagger 2.0 specification.

Light-4j encourages design driven implementation so Swagger specification should be done before the implementation starts. With the [light-codegen][] generator, the server stub can be generated and start running within seconds. However, we cannot rely on the generator for validation as specification will be changed along the life cycle of the API. This is why we have provided a validator that works on top of the specification at runtime. In this way, the generator should only be used once, and the validator will take the latest specification and validate according to the specification at runtime.

### Fail fast

As you may notice, our Status object only supports one error code and message. This is the indication the framework validation is designed to fail fast. Whenever there is an error, the server will stop processing the request and return the error to the consumer immediately. There are several reasons for this design, and they are documented in [fail-fast][] in the design section. 

### ValidatorHandler

This is the entry point of the validator, and it is injected during server startup if the `swagger-validator.yml` enabled is true. By default, only 
RequestValidator will be called. Response validation should be done in the [client][] module. 

### Configuration

From release 1.5.18, Light supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a validator configuration file for different frameworks, a new `swagger-validator.yml` with the same content as the old `validator.yml` has been introduced. The validator.yml is still loaded if swagger-validator.yml doesn’t exist for backward compatibility.

Here is an example of swagger-validator.yml

```
# This is specific Swagger validator configuration file. It is introduced to support multiple
# frameworks in the same server instance and it is recommended. If this file cannot be found,
# the generic validator.yml will be loaded as a fallback.
---
# Enable request validation. Response validation is not done on the server but client.
enabled: true
# Log error message if validation error occurs
logError: true
# Skip body validation set to true if used in light-router, light-proxy and light-spring-boot.
skipBodyValidation: false

```

As we know the `swagger-validator` can validate the request body if the body is parsed by the body handler. For put, post and patch requests, if the body is missing from the exchange attachment, an error message that indicates the body is missing will be returned. There are two different situations in which we cannot use the body parser in the request chain as once the body stream is consumed, it won’t be available for the downstream handlers anymore.

* Validation in [light-router][] and [light-proxy][]. We have to keep the body stream to pass to the downstream service so we cannot consume it.

* In [light-spring-boot][] application. We have to keep the body so that Spring Boot can consume it.

To ensure that the two situations are handled correctly, we can set the skipBodyValidation to true in the above config file. If the body object cannot be found in the exchange(which means the body parser is not wired in) and the skipBodyValidation is true, then we skip the body validation in the validator. 

This flag is set to false as default for backward compatibility and your existing `swagger-validator.yml` still works without this flag set. 

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
[light-router]: /service/router/
[light-proxy]: /service/proxy/
[light-spring-boot]: /style/light-spring-boot/
