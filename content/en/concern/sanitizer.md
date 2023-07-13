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

This is a middleware handler that addresses cross-site scripting concerns. It encodes the header and body according to the configuration. As body encoding depends on [BodyHandler][] middleware, it has to be plugged into the request/response chain after the body handler if the sanitizer.bodyEnabled is true in the values.yml

If you use the http-sidecar or light-gateway, the sanitizer handler will only work with the headers. The body sanitization won't work as the request body won't be intercepted but transferred to the backend API directly. The only option to sanitize the request body is to set up a transform rule with a plugin to transform the request body on the sidecar or gateway. For more details on utilizing the body-sanitizer plugin, please visit [here](https://github.com/networknt/yaml-rule-plugin/tree/master/body-sanitizer).

### Configuration

Here is the default configuration file sanitizer.yml

```
---
# Sanitize request for cross-site scripting during runtime

# indicate if sanitizer is enabled or not
enabled: ${sanitizer.enabled:false}

# if it is enabled, the body needs to be sanitized
bodyEnabled: ${sanitizer.bodyEnabled:true}
# the encoder for the body. javascript, javascript-attribute, javascript-block or javascript-source
# There are other encoders that you can choose depending on your requirement. Please refer to site
# https://github.com/OWASP/owasp-java-encoder/blob/main/core/src/main/java/org/owasp/encoder/Encoders.java
bodyEncoder: ${sanitizer.bodyEncoder:javascript-source}
# pick up a list of keys to encode the values to limit the scope to only selected keys. You can
# choose this option if you want to only encode certain fields in the body. When this option is
# selected, you can not use the bodyAttributesToIgnore list.
bodyAttributesToEncode: ${sanitizer.bodyAttributesToEncode:}
# pick up a list of keys to ignore the values encoding to skip some of the values so that these
# values won't be encoded. You can choose this option if you want to encode everything except
# several values with a list of the keys. When this option is selected, you can not use the
# bodyAttributesToEncode list.
bodyAttributesToIgnore: ${sanitizer.bodyAttributesToIgnore:}

# if it is enabled, the header needs to be sanitized
headerEnabled: ${sanitizer.headerEnabled:true}
# the encoder for the header. javascript, javascript-attribute, javascript-block or javascript-source
# There are other encoders that you can choose depending on your requirement. Please refer to site
# https://github.com/OWASP/owasp-java-encoder/blob/main/core/src/main/java/org/owasp/encoder/Encoders.java
headerEncoder: ${sanitizer.headerEncoder:javascript-source}
# pick up a list of keys to encode the values to limit the scope to only selected keys. You can
# choose this option if you want to only encode certain values in the headers. When this option is
# selected, you can not use the headerAttributesToIgnore list.
headerAttributesToEncode: ${sanitizer.headerAttributesToEncode:}
# pick up a list of keys to ignore the values encoding to skip some of the values so that these
# values won't be encoded. You can choose this option if you want to encode everything except
# several values with a list of the keys. When this option is selected, you can not use the
# headerAttributesToEncode list.
headerAttributesToIgnore: ${sanitizer.headerAttributesToIgnore:}
```

As the enabled default is false, this middleware won't be loaded during server startup. 

Once the handler is enabled, both bodyEnabled and headerEnabled are true. If you are using it on light-gateway or http-sidecar, please make sure that bodyEnabled is false. You need to use the body-sanitizer plugin to encode/transform the request body before it is transferred to the downstream API. 


### When to use Sanitizer 

This handler should only be used when collecting user input from Web/Mobile UI and, later on, using the input data to generate web pages, for example, a forum or blog application.

Don't use this handler for services where user input will never be used to construct web pages.

### Query Parameters

In other platforms, especially JEE containers, query parameters must also be sanitized. However, I have found that Undertow does sanitize special characters in query parameters. This is why this handler doesn't do anything about query parameters.

### Encode Library

The library used for cross-site scripting sanitization is from https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)
and the library can be found at https://github.com/OWASP/owasp-java-encoder

### Encode Level

The encoding level we are using for both header and body is forJavaScriptSource. It gives us a certain level of confidence, and it won’t mess up the header and body in most cases.

[BodyHandler]: /concern/body/
