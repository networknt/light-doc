---
title: "Mask"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
---

In the entire life cycle of the exchange, there might a lot of logging statements written to log files or other persistence storage. These logs will be used to assist production issue identifying and resolving, and a broad group of people might have access to these logs. For confidentiality, sensitive info needs to be masked before logging, for example, credit card number, sin number, etc.

### StarupHookProvider

The mask module depends on JsonPath which is a third party library that gives us access elements in JSON string easily. If the mask module is used, you should specify the JSON Parser for the JsonPath during the server startup to switch from the default json-smart parser to the parser you are using within your application. In most of the cases, it is Jackson which is the default in light-4j. 

In the server module, we have provided a JsonPathStartupHookProvider class that you can use out of the box. Just update the service.yml to add one more StartupHookProvider. 

As we don't know if the mask module is used or not in your application, we will add this JsonPathStartupHookProvider to the generated service.ym in the light-codegen but comment it out. In this way, it would be easy for the end user to enable it if they want to leverage the mask module. 

in service.yml
```
singletons:
- com.networknt.server.StartupHookProvider:
    - com.networknt.server.JsonPathStartupHookProvider
```

### Configuration

Given different API will have different sensitive data, the mask is configurable and can be applied at the header, cookie, query parameters, and body.

### Mask with String

### Mask with Regex

### Mask with JsonPath

### Mask with Map and List

