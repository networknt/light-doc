---
title: "Consul Connection Overflow"
date: 2018-10-31T00:36:49-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

If you saw the following error, chances are you need to increase the bufferSize in your client.yml file. The default is 24 which is 24k bytes. It should be OK for small applications, but for bigger request body, you need to increase it to 100 or even 1024 for one of our customers. 

```
java.io.IOException: overflow
 	at io.undertow.protocols.ssl.SslConduit.doWrap(SslConduit.java:903)
 	at io.undertow.protocols.ssl.SslConduit.doUnwrap(SslConduit.java:671)
 	at io.undertow.protocols.ssl.SslConduit.read(SslConduit.java:567)
 	at org.xnio.conduits.ConduitStreamSourceChannel.read(ConduitStreamSourceChannel.java:127)
 	at io.undertow.client.ALPNClientSelector$2.handleEvent(ALPNClientSelector.java:87)
 	at io.undertow.client.ALPNClientSelector$2.handleEvent(ALPNClientSelector.java:77)
 	at org.xnio.ChannelListeners.invokeChannelListener(ChannelListeners.java:92)
 	at org.xnio.conduits.ReadReadyHandler$ChannelListenerHandler.readReady(ReadReadyHandler.java:66)
 	at io.undertow.protocols.ssl.SslConduit$SslReadReadyHandler.readReady(SslConduit.java:1167)
 	at io.undertow.protocols.ssl.SslConduit$SslWriteReadyHandler.writeReady(SslConduit.java:1242)
 	at org.xnio.nio.NioSocketConduit.handleReady(NioSocketConduit.java:93)
 	at org.xnio.nio.WorkerThread.run(WorkerThread.java:561)
```

