---
title: "Body Parser"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
---

Body parser is a middleware handler designed for [light-rest-4j][]only. It will parse 
the body to a list or map according to the content-type in the HTTP header for POST, 
PUT and PATCH HTTP methods. The content-type can be parsed includes application/json,
multipart/form-data and application/x-www-form-urlencoded. The application/json type is 
parsed into list or map depending on the first character of the body while the others 
is parsed into map by using FormParserFactory provided by io.undertow. After the body 
is parsed, it will be attached to the exchange so that subsequent handlers can use it 
directly. 

### Introduction

In order for this handler to work, the content-type in header must be started with 
"application/json", "multipart/form-data" or "application/x-www-form-urlencoded". 
If the content type is correct, it will parse it to List or Map and put it into 
REQUEST_BODY exchange attachment.

If content type is missing or if it is not started as "application/json", "multipart/form-data" 
or "application/x-www-form-urlencoded". the bodywon't be parsed and this handler will 
just call next handler in the chain. 

[Sanitizer][], [Openapi Validator][] and [Swagger Validator][] depend on this middleware.

### Configuration

Here is an example of configuration body.yml which has a flag to enable or disable it.

```
# Enable body parse flag
enabled: true
```

### Parser

Here is the parsing logic. 

```java
                    if (contentType != null && contentType.startsWith("application/json")) {
                        if (exchange.isInIoThread()) {
                            exchange.dispatch(this);
                            return;
                        }
                        exchange.startBlocking();
                        InputStream is = exchange.getInputStream();
                        try {
                            Object body;
                            String s = StringUtils.inputStreamToString(is, StandardCharsets.UTF_8);
                            if (s != null) {
                                s = s.trim();
                                if (s.startsWith("{")) {
                                    body = Config.getInstance().getMapper().readValue(s, new TypeReference<HashMap<String, Object>>() {});
                                } else if (s.startsWith("[")) {
                                    body = Config.getInstance().getMapper().readValue(s, new TypeReference<List<Object>>() {});
                                } else {
                                    // error here. The content type in head doesn't match the body.
                                    setExchangeStatus(exchange, CONTENT_TYPE_MISMATCH, contentType);
                                    return;
                                }
                                exchange.putAttachment(REQUEST_BODY, body);
                            }
                        } catch (IOException e) {
                            logger.error("IOException: ", e);
                            setExchangeStatus(exchange, CONTENT_TYPE_MISMATCH, contentType);
                            return;
                        }
                        // parse the body to form-data if content type is multipart/form-data or application/x-www-form-urlencoded
                    } else if (contentType != null &&
                            (contentType.startsWith("multipart/form-data") || contentType.startsWith("application/x-www-form-urlencoded"))) {
                        if (exchange.isInIoThread()) {
                            exchange.dispatch(this);
                            return;
                        }
                        exchange.startBlocking();
                        try {
                            Object data;
                            FormParserFactory formParserFactory = FormParserFactory.builder().build();
                            FormDataParser parser = formParserFactory.createParser(exchange);
                            if (parser != null) {
                                FormData formData = parser.parseBlocking();
                                data = BodyConverter.convert(formData);
                                exchange.putAttachment(REQUEST_BODY, data);
                            }
                        } catch (Exception e) {
                            logger.error("IOException: ", e);
                            setExchangeStatus(exchange, CONTENT_TYPE_MISMATCH, contentType);
                            return;
                        }
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
[Openapi Validator]: /style/light-rest-4j/openapi-validator/
[Swagger Validator]: /style/light-rest-4j/swagger-validator/
