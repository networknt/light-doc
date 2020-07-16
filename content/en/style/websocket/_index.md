---
title: "Websocket with light platform"
date: 2017-12-20T21:21:31-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
---

Light-4j is built on top of Undertow core, and it supports Websocket natively without going through Java EE stack. So the performance is much better than other servers in terms of throughput and latency. There are two examples in [light-example-4j][] to demo client-to-server and peer-to-peer WebSocket. 

Both projects have been upgraded to the light-4j 2.0.x with Java 11 support. To build and run them, run the following command lines. 

### client-to-server

```
cd ~/networknt/light-example-4j/websocket/client-to-server
mvn clean install exec:exec
```

Open a browser and paste the address http://localhost:8080 and you can send WebSocket message to the server. Once the server receives the message, it will respond with a popup window. 


### peer-to-peer

This is a chat server that supports two or more browser sessions to send messages to each other.


```
cd ~/networknt/light-example-4j/websocket/peer-to-peer
mvn clean install exec:exec
```

Open two different browsers or two tabs from one browser and send messages to see if all messages are shown up. 



[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/websocket
