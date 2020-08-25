---
title: "Body Parser"
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

Body parser is a middleware handler designed for [light-rest-4j][]only. It will parse the body to a list or map according to the content-type in the HTTP header for POST, PUT and PATCH HTTP methods. The content-type can be parsed includes "application/json", "text/plain", "multipart/form-data" and "application/x-www-form-urlencoded". The "application/json" type is parsed into a list or map depending on the first character of the body. The "text/plain" will be parsed into a String object. The multipart/form-data" and "application/x-www-form-urlencoded" will be parsed to a map of fields. Other content-types will be handled as an input stream, including content-type is missing from the HTTP header. 

After the body is parsed, it will be attached to the exchange so that subsequent handlers can use it directly from the exchange attachment.

[Sanitizer][] and [OpenApi Validator][] depend on this middleware.

### Configuration

Here is an example of configuration body.yml which has a flag to enable or disable it.

```
# Enable body parse flag
enabled: true
```

### Parser

Here is the parsing logic. 

```java
        if (contentType != null) {
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
                return;
            }
        } else {
            // attach the stream to the exchange if the content type is missing.
            InputStream inputStream = exchange.getInputStream();
            exchange.putAttachment(REQUEST_BODY, inputStream);
        }
```
### Example

Here is the code example that get the boby object(Map or List) from exchange.

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
