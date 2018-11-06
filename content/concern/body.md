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
the body to a list or map depending on the first character of the body content if 
application/json is the content-type in the HTTP header for POST, PUT and PATCH HTTP 
methods. After the body is parsed, it will be attached to the exchange so that subsequent 
handlers can use it directly. In the future, other content-type might be supported if needed.

### Introduction

In order for this handler to work, the content-type in header must be started with 
"application/json". If the content type is correct, it will parse it to List or Map 
and put it into REQUEST_BODY exchange attachment.

If content type is missing or if it is not started as "application/json", the body
won't be parsed and this handler will just call next handler in the chain. 

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
                        String s = new Scanner(is, "UTF-8").useDelimiter("\\A").next();
                        s = s.trim();
                        if (s.startsWith("{")) {
                            body = Config.getInstance().getMapper().readValue(s, new TypeReference<HashMap<String, Object>>() {
                            });
                        } else if (s.startsWith("[")) {
                            body = Config.getInstance().getMapper().readValue(s, new TypeReference<List<Object>>() {
                            });
                        } else {
                            // error here. The content type in head doesn't match the body.
                            Status status = new Status(CONTENT_TYPE_MISMATCH, contentType);
                            exchange.setStatusCode(status.getStatusCode());
                            exchange.getResponseSender().send(status.toString());
                            return;
                        }
                        exchange.putAttachment(REQUEST_BODY, body);

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
