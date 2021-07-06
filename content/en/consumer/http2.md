---
title: "HTTP 2.0"
date: 2018-03-01T21:37:12-05:00
description: ""
categories: []
keywords: []
weight: 30
aliases: []
toc: false
draft: false
reviewed: true
---

By default, the services built on top of Light have HTTP 2.0 enabled automatically right of the light-codegen. The server also supports HTTP 1.1 connection if the client cannot negotiate HTTP 2.0 with the server. 

There are a lot of benefits with HTTP 2.0, but JDK 8 and below don't support it natively. Though there are some workarounds, none of them is a practical solution. Given the situation, we have built a client module based on Undertow core for HTTP 2.0 support. 

Here are some links regarding HTTP 2.0 and HTTP 1.1

https://www.pacwebhosting.co.uk/insight/News/NewsItem/View/31/http11-vs-http2-whats-the-difference

https://imagekit.io/demo/http2-vs-http1

https://www.upwork.com/hiring/development/the-http2-protocol-its-pros-cons-and-how-to-start-using-it/

### Server Config

To enable HTTP 2.0 on the server side, you need to ensure that enableHttp2 is true in server.yml

Please refer to [server module] and server.yml configuration for more info.

### Client Module

When using the light-4j client module to make an HTTP request to the server, the HTTP 2.0 is not enabled by default. You have to specify you want to establish an HTTP 2.0 connection explicitly. 

Here is the code example that creates a connection. 

```java
        if(connection == null || !connection.isOpen()) {
            try {
                connection = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
```

The option below will create an HTTP 2.0 connection. 

```java

OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)

```

If you are sure that the service is built on light-4j and HTTP 2.0 is not disabled. Then you should use HTTP 2.0 from the consumer side to enjoy the fast speed and single connection multiplexing. 

### Other Java HTTP Client

Most Java HTTP Client doesn't support HTTP 2.0, or you have to find a workaround to do that with bootclasspath jar file per version of JDK on Java 8. If you are using Java 11, then it has native support for HTTP 2.0; however, it is never been tested with the foundation framework. 

[server module]: /concern/server/
