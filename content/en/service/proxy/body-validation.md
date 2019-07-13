---
title: "Body Validation"
date: 2019-07-13T10:25:33-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

As we know, you can plug in over 100 middleware handlers from the light platform in the light-proxy to customize it for your needs. One of the features most users are interested in is the schema validation on the proxy. For example, when you have the OpenAPI 3.0 or Swagger 2.0 specification for the backend service, you can deploy the swagger.json or openapi.yaml to the proxy and wire in the swagger-validator or openapi-validator to perform the validation against the specification. 

For PUT/POST/PATCH requests that pass through the proxy, there are some special configuration considerations. 

Before swagger-validator or openapi-validator handler, the body handler must be in the request chain to parse the HTTP body into a JSON object. This will consume the HTTP body stream (need to serialize and deserialize) which is very time-consuming on the proxy. The same stream, of course, will be deserialized on the backend service again for the processing. To avoid this performance issue, we recommend the following configuration. 

### Body Handler

Don't plug in the body parser handler at all in the proxy so that the HTTP stream can be forwarded to the backend service without deserialization and serialization. 

If you are using the handler.yml for the middleware configuration, you can remove the body handler from all the POST/PUT/PATCH paths. 

### Validation Handler

Both Swagger and OpenAPI validation handler will rely on the body handler to parse the request body and won't work if the body object doesn't exist in the exchange as an attachment. 

To bypass the body validation on the proxy, we can disable the body validation only in the swagger-validator.yml or openapi-validator.yml

Here is an example for openapi-validator.yml

```
# Default openapi-validator configuration
---
# Enable request validation. Response validation is not done on the server but client.
enabled: true
# Log error message if validation error occurs
logError: true
# Skip body validation set to true if used in light-router, light-proxy and light-spring-boot.
skipBodyValidation: false
# Enable response validation.
validateResponse: false
```

You need to add this config file to your proxy config folder and change the skipBodyValidation to true. 

Once the body handler is removed, and skipBodyValidation is set to true, the request pass through the proxy will be validated against the specification for headers, query parameters, path parameters but not the request body. With this setup, the backend service will be responsible for validating the body. 


