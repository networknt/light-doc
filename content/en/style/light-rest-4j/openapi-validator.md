---
title: "OpenAPI Validator"
date: 2017-11-22T22:46:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This handler is part of the [light-rest-4j][] which is built on top of light-4j but focused on RESTful API only. Also, it only works with the OpenAPI 3.0 specification.

Light-4j encourages design driven implementation so OpenAPI specification should be done before the implementation starts. With the [light-codegen][] generator, the server stub can be generated and start running within seconds. However, we cannot rely on the generator for validation as specification will be changed along the life cycle of the API. This is why we have provided a validator that works on top of the specification at runtime. In this way, the generator should only be used once, and the validator will take the latest specification and validate according to the specification at runtime. 

### Fail fast

As you may notice, our Status object only supports one error code and message. This is the indication the framework validation is designed to fail fast. Whenever there is an error, the server will stop processing the request and return the error to the consumer immediately. There are several reasons for this design, and they are documented in [fail-fast][] in the design section. 

### ValidatorHandler

It is the entry point of the validator, and it is injected during server startup if openapi-validator.yml enabled is true. By default, only RequestValidator will be called. Response validation should be done in the [client][] module. 

### Configuration

From release 1.5.18, Light supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a validator configuration file for different frameworks, a new `openapi-validator.yml` with the same content as the old `validator.yml` has been introduced. The `validator.yml` is still loaded if `openapi-validator.yml` doesn't exist for backward compatibility. 

Here is an example of `openapi-validator.yml`

```
# This is specific OpenAPI validator configuration file. It is introduced to support multiple
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

As we know the `openapi-validator` can validate the request body if the body is parsed by the body handler. For put, post and patch request, if the body is missing from the exchange attachment, an error message that indicates the body is missing will be returned. There are two different situations that we cannot use the body parser in the request chain as once the body stream is consumed, it won't be available for the downstream handlers anymore.

* Validation in [light-router][] and [light-proxy][]. We have to keep the body stream to pass to the downstream service so we cannot consume it.

* In [light-spring-boot][] application. We have to keep the body so that Spring Boot can consume it.

To ensure that the two situations are handled correctly, we can set the skipBodyValidation to true in the above config file. If the body object cannot be found in the exchange(which means the body parser is not wired in) and the skipBodyValidation is true, then we skip the body validation in the validator. 

This flag is set to false as default for backward compatibility and your existing `openapi-validator.yml` still works without this flag set. 

### RequestValidator

It will validate the following:

* uri
* method
* header
* query parameters
* path parameters
* body if available

When necessary, [json-schema-validator][] will be called to do json schema validation.

### ResponseValidator
ResponseValidator currently can only be used on server side, because it needs to load the related specification, which is loaded by .
ResponseValidator should be able to validate:
* Response Header (not support yet)
* Response Body

To validate response content:  
ResponseValidator can validate a given response content object with schema coordinate (uri, httpMethod, statusCode, mediaTypeName)  
Based on OpenAPI specification, the [Response Object][] is located in Path Object -> Operation Object -> Responses-> Response Object. For example:
```
paths:
  /todoItems:
    get:
      description: return all to-do items
      operationId: getTodoItemList
      responses:
        '200':
          description: all todo items response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/TodoItem/'
components:
  schemas:
    TodoItem:
      type: object
      required:
        - id
        - content
      properties:
        id:
          type: string
        content:
          type: string
```
Thus, to locate a response schema, we need to know the path: "/todoItems", the operation method: "get", the corresponding status code: "200", and the content media type: "application/json".  
In TodoItemsGetHandler(Auto-generated by light-codegen)
```
    ResponseValidator validator = new ResponseValidator();
    Status status = validator.validateResponseContent(body, "/todoItems", "get");
    if(status != null) {
        exchange.getResponseSender().send("{\"code\" : \"error\"}");
    }
    exchange.getResponseSender().send(body);
```
The status code and content media type is **optional**, if not specify status code, it will be default to 200. if not specify content media type, it should be default to "application/json"
Result:  
when the response body is:
```
[
   {
   "id": 234,
   "content": "content1"
   },
   {
   "id": "abcde",
   "content": "content2"
   }
]
```
the actual response is:
```
{"code" : "error"}
```
when the response body is:
```
[
   {
   "id": "234",
   "content": "content1"
   },
   {
   "id": "abcde",
   "content": "content2"
   }
]
```
the actual response is:
```
[
   {
   "id": "234",
   "content": "content1"
   },
   {
   "id": "abcde",
   "content": "content2"
   }
]
```

##### For testing usage:
Test cases for validating each endpoint's response can be generated by light-codegen.
To do it manualy, you need to get response body, and status code, content-type. Then all you need to do is to call ResponseValidator.validateResponseContent()
An example code:
```
    String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
    Optional<HeaderValues> contentTypeName = Optional.ofNullable(reference.get().getResponseHeaders().get(Headers.CONTENT_TYPE));
    SchemaValidatorsConfig config = new SchemaValidatorsConfig();
    config.setMissingNodeAsError(true);
    ResponseValidator responseValidator = new ResponseValidator(config);
    int statusCode = reference.get().getResponseCode();
    Status status;
    if(contentTypeName.isPresent()) {
        status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), contentTypeName.get().getFirst());
    } else {
        status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), JSON_MEDIA_TYPE);
    }
```
##### SchemaValidatorsConfig:
You may customize ResponseValidator by set values in SchemaValidatorsConfig.
* **typeLoose**: this is the switcher to enable if validator try to convert a String to a corresponding type. for example, it will try to convert a String "123" to a int 123. When validate response body, it is set to false all the time.
* **missingNodeAsError**:  
* **elementValidationError**: 

### SchemaValidator

If a schema is defined in openapi.yaml or openapi.json, then the [json-schema-validator][] will be called to validate the input against a JSON schema defined in draft v4.

### Test

In order to test validator, the test suite starts a light-4j server and serves the petstore api for testing. It is a demo on how to unit test your API during development.

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[light-codegen]: /tool/light-codegen/
[fail-fast]: /architecture/fail-fast/
[client]: /concern/client/
[json-schema-validator]: https://github.com/networknt/json-schema-validator
[light-router]: /service/router/
[light-proxy]: /service/proxy/
[light-spring-boot]: /style/light-spring-boot/
[Response Object]: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#response-object
