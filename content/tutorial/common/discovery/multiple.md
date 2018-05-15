---
title: Multiple Instances of the same Service
linktitle: Multiple Instances of the same Service
date: 2017-10-17T19:08:51-04:00
lastmod: 2018-05-15
description: "Client side load balancing through direct service discovery 
using configuration."
weight: 40
sections_weight: 40
draft: false
toc: true
---

## Introduction

In this step we're going to start two `API D` instances that will listen to `7444` and `7445`.

Let's copy our current state from last step into the `multiple` directory.

```bash
cp -r ~/networknt/discovery/api_a/dynamic/ ~/networknt/discovery/api_a/multiple
cp -r ~/networknt/discovery/api_b/dynamic/ ~/networknt/discovery/api_b/multiple
cp -r ~/networknt/discovery/api_c/dynamic/ ~/networknt/discovery/api_c/multiple
cp -r ~/networknt/discovery/api_d/dynamic/ ~/networknt/discovery/api_d/multiple
```

## Configuring the Servers
 
### API B 

Let's modify API B service.yml to have two API D instances that listen to 7444
and 7445.

```yaml
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      parameters:
        com.networknt.apid-1.0.0: https://localhost:7444,https://localhost:7445
```

Also, so that we can see different end points being hit, lets disable the connection
caching in the request to `API D`.

Change the following section in `DataGetHandler.java`:
```java
if(connection == null || !connection.isOpen()) {
    connection = getConnectionD();
}
```

to:

```java
connection = getConnectionD();
```

### API D

In order to start two instances with the same code base, we need to modify the
server.yml before starting the second server. 

Also, let's update the handler so that we know which port serves the request.

`DataGetHandler.java`

```java
package com.networknt.apid.handler;

import com.networknt.config.Config;
import com.networknt.server.Server;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;

import java.util.ArrayList;
import java.util.List;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        int port = Server.config.getHttpsPort();
        List<String> messages = new ArrayList<>();
        messages.add("API D: Message 1 from port " + port);
        messages.add("API D: Message 2 from port " + port);
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}

```

## Starting the Servers

Now let's start all five servers from five terminals. API D has two instances.

**API A**

```
cd ~/networknt/discovery/api_a/multiple
mvn clean install exec:exec
```

**API B**

```
cd ~/networknt/discovery/api_b/multiple
mvn clean install exec:exec

```

**API C**

```
cd ~/networknt/discovery/api_c/multiple
mvn clean install exec:exec

```

**API D**


And start the first instance that listen to 7444.

```
cd ~/networknt/discovery/api_d/multiple
mvn clean install exec:exec

```
 
Now let's start the second instance of `API D`. Before starting the serer, let's update
server.yml with https port 7445 and disable HTTP.

```yaml
httpsPort: 7445
```

And start the second instance

```
cd ~/networknt/discovery/api_d/multiple
mvn clean install exec:exec

```

## Testing the Servers

```
curl -k https://localhost:7441/v1/data
```

And the result can be from port 7444 or 7445 as Round Robin load balancer will pick up on of them.

```
["API D: Message 1 from port 7445","API D: Message 2 from port 7445","API C: Message 1","API C: Message 2"]
```

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API C: Message 1","API C: Message 2"]
```

The next step, we are going to use consul to do the service registry and discovery.

[Dynamic]: /tutorial/common/discovery/dynamic/