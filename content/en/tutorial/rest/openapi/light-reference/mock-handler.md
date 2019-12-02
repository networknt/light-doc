---
title: "Mock Handler"
date: 2019-12-01T23:09:51-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have the project [generated][] in the light-reference folder. In this step, we are going to update the generated RdataHostGetHandler.java to mock the output. 



```
package com.networknt.reference.handler;

import com.networknt.handler.LightHttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class RdataHostGetHandler implements LightHttpHandler {
    
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        String host = exchange.getQueryParameters().get("host").getFirst();
        String name = exchange.getQueryParameters().get("name").getFirst();
        String result = "{\"key\":\"value\"}";
        if("com.networknt".equals(host)) {
            if("role".equals(name)) {
                result = "[{\"admin\":\"admin\"},{\"user\":\"user\"}]";
            } else if("client".equals(name)) {
                result = "[{\"trusted\":\"trusted\"},{\"private\":\"private\"},{\"public\":\"public\"}]";
            }
        }
        exchange.getResponseSender().send(result);
    }
}

```

Build and start the server from the terminal. 

```
mvn clean install exec:exec
```

From another terminal, issue the command.

```
curl -k https://localhost:8443/rdata/com.networknt?name=role
```

The result will be. 

```
[{"admin":"admin"},{"user":"user"}]
```

This means that the service is working. In the next step, we are going to [deploy][] it to the test cloud. 

[generated]: /tutorial/rest/openapi/light-reference/generation/
[deploy]: /tutorial/rest/openapi/light-reference/docker-deploy/
