---
title: "Sanitizer"
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

This is a middleware that addresses cross-site scripting concerns. It encodes the header and body according to the configuration. As body encoding depends on [Body][] middleware, it has to be plugged into the request/response chain after Body.

### Configuration

Here is the default configuration file sanitizer.yml

```
# Sanitize request for cross site scripting during runtime

# indicate if sanitizer is enabled or not
enabled: true

# if it is enabled, does body need to be sanitized
sanitizeBody: true

# if it is enabled, does header need to be sanitized
sanitizeHeader: false
```

If the enabled flag is false, this middleware won't be loaded during server startup. 

sanitizeBody and sanitizeHeader control if the body and/or the header need to be sanitized. In most of the cases, sanitizing the body makes sense and sanitizing the header is not necessary.

### When to use Sanitizer 

This handler should only be used when you are collecting user input from Web/Mobile UI and later on, using the input data to generate web pages, for example, a forum or blog application.

For services where user input will never be used to construct web pages, don’t use this handler.

### Query Parameters

In other platforms especially JEE containers, query parameters need to be sanitized as well. However, I have found that Undertow does sanitize special characters in query parameters. This is why this handler doesn't do anything about query parameters.

### Encode Library

The library used for cross-site scripting sanitization is from https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)
and the library can be found at https://github.com/OWASP/owasp-java-encoder

### Encode Level

The encoding level we are using for both header and body is forJavaScriptSource. It gives us a certain level of confidence, and it won’t mess up the header and body in most cases.

[Body]: /concern/body/
