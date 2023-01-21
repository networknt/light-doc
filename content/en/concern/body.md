---
title: "Body Handler"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

### Introduction

Body parser is a middleware handler designed for [light-rest-4j][] only. It will parse the body to a list or map according to the content-type in the HTTP header for POST, PUT and PATCH HTTP methods. The content-type that can be parsed includes “application/json”, “text/plain”, “multipart/form-data'' and “application/x-www-form-urlencoded”. The “application/json” type is parsed into a list or map depending on the first character of the body. The “text/plain” will be parsed into a String object. The multipart/form-data'' and “application/x-www-form-urlencoded” will be parsed to a map of fields. Other content-types will be handled as an input stream, including content-type which is missing from the HTTP header.

After the body is parsed, it will be attached to the exchange so that subsequent handlers can use it directly from the exchange attachment.

[Sanitizer][] and [OpenApi Validator][] depend on this middleware.

The purpose of the BodyHandler is to convert the request body stream to an easy-to-use Java object so that all subsequent handlers can share the same thing. As the request body stream can only be consumed once, it is easy for us to provide this standard handler to do that job in the request/response chain. 

Also, we need to be aware that this handler is not suitable in the http-sidecar and the light-gateway as the request body stream needs to be transferred to the downstream API and cannot be consumed. For the sidecar and the gateway, we should use [RequestBodyInterceptor][] and [ResponseBodyInterceptor][] instead.

### Configuration

Here is a code example that receives the body object (Map or List) from the exchange.

```
# Enable body parse flag
enabled: ${body.enabled:true}
# cache request body as a string along with JSON object. The string formatted request body will be used for audit log.
# you should only enable this if you have configured audit.yml to log the request body as it uses extra memory.
cacheRequestBody: ${body.cacheRequestBody:false}
# cache response body as a string along with JSON object. The string formatted response body will be used for audit log.
# you should only enable this if you have configured audit.yml to log the response body as it uses extra memory.
cacheResponseBody: ${body.cacheResponseBody:false}
```

### Parser

Here is the parsing logic. 

```java
    @Override
    public void handleRequest(final HttpServerExchange exchange) throws Exception {
        if(logger.isDebugEnabled()) logger.debug("BodyHandler.handleRequest starts.");
        // parse the body to map or list if content type is application/json
        String contentType = exchange.getRequestHeaders().getFirst(Headers.CONTENT_TYPE);
        if (contentType != null) {
            if (exchange.isInIoThread()) {
                exchange.dispatch(this);
                return;
            }
            exchange.startBlocking();
            try {
                if (contentType.startsWith("application/json")) {
                    InputStream inputStream = exchange.getInputStream();
                    String unparsedRequestBody = StringUtils.inputStreamToString(inputStream, StandardCharsets.UTF_8);
                    // attach the unparsed request body into exchange if the cacheRequestBody is enabled in body.yml
                    if (config.isCacheRequestBody()) {
                        exchange.putAttachment(REQUEST_BODY_STRING, unparsedRequestBody);
                    }
                    // attach the parsed request body into exchange if the body parser is enabled
                    attachJsonBody(exchange, unparsedRequestBody);
                } else if (contentType.startsWith("text/plain")) {
                    InputStream inputStream = exchange.getInputStream();
                    String unparsedRequestBody = StringUtils.inputStreamToString(inputStream, StandardCharsets.UTF_8);
                    exchange.putAttachment(REQUEST_BODY, unparsedRequestBody);
                } else if (contentType.startsWith("multipart/form-data") || contentType.startsWith("application/x-www-form-urlencoded")) {
                    // attach the parsed request body into exchange if the body parser is enabled
                    attachFormDataBody(exchange);
                } else {
                    InputStream inputStream = exchange.getInputStream();
                    exchange.putAttachment(REQUEST_BODY, inputStream);
                }
            } catch (IOException e) {
                logger.error("IOException: ", e);
                setExchangeStatus(exchange, CONTENT_TYPE_MISMATCH, contentType);
                if(logger.isDebugEnabled()) logger.debug("BodyHandler.handleRequest ends with an error.");
                return;
            }
        }
        if(logger.isDebugEnabled()) logger.debug("BodyHandler.handleRequest ends.");
        Handler.next(exchange, next);
    }
```
### Example

Here is the code example that gets the body object(Map or List) from the exchange in subsequent middleware handlers or business handlers.

```java
                    Object body = exchange.getAttachment(BodyHandler.REQUEST_BODY);
                    if(body == null) {
                        exchange.getResponseSender().send("nobody");
                    } else {
                        if(body instanceof List) {
                            exchange.getResponseSender().send("list");
                        } else {
                            exchange.getResponseSender().send("map");
                        }
                    }

```

[light-rest-4j]: /style/light-rest-4j/
[Sanitizer]: /concern/sanitizer/
[OpenApi Validator]: /style/light-rest-4j/openapi-validator/
[RequestBodyInterceptor]: /concern/request-body-interceptor/
[ResponseBodyInterceptor]: /concern/response-body-interceptor/
