---
title: "Response Encode and Request Decode"
date: 2018-11-06T15:21:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

To reduce bandwidth usage and speed up the delivery, many clients and servers utilize `gzip` or `deflate` encoding for large requests or responses.

One of our users built an API that receives batch data from iOS, and the request is encoded with gzip. It requires that we have a RequestDecodeHandler before the BodyHandler to decode the gzip stream so that subsequent handlers can handle the request as a normal JSON body.

Sometimes, we might have services that produce big size JSON responses to the client, and we might consider encoding the response body with gzip as the majority of browsers support it.

This module contains two middleware handlers one works on the incoming request and the other works on the outgoing response. Usually, there is only one handler that should be used by a particular service.

### RequestDecodeHandler

This middleware handler must be wired in the request handler chain before BodyHandler. It looks for the header for encoding and decoding the gzip and deflating the stream to normal JSON for subsequent handlers to process.

A typical request will have headers like this. 

```
{content-encoding=[gzip], accept-encoding=[gzip], content-type=[application/json] ...
```

If your service has clients sending both encoded or non-encoded requests, you can safely wire in this handler as it will check the header and decode when necessary. 

After wiring in this handle in service.yml or handler.yml, you need to add a configuration file request-decode.yml to the config folder.

Here is the content for most applications that need this handler. 


```
---
# request decode handler for gzip and deflate
enabled: true
decoders:
  - gzip
  - deflate

```

### ResponseEncodeHandler

This middleware handler can be wired in anywhere in the chain as it works on the response only. It looks for the header for the encoding requirement and picks up one of the providers: gzip or deflate.

The client request must have a header called accept-encoding=gzip or accept-encoding=deflate. 

If requests doen't contain above header, then the normal stream will be sent. 

After wiring in this handler in service.yml or handler.yml, you need to add a configuration file response-encode.yml to the config folder.

Here is the content for most applications that need this handler.

```
---
# response encode handler for gzip and deflate
enabled: true
encoders:
  - gzip
  - deflate
```


