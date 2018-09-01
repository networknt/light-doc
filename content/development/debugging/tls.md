---
title: "Debug TLS Handshake"
date: 2018-09-01T09:23:07-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When you have service to service communications between services, HTTPS is a must and we have both client and server [keystore and truststore][] to support one-way TLS or two-way TLS. 

For most developers, manipulating keystore and truststore with [keytool][] would be new for them. If it is not working, turn on the network debug would help greatly. 

Just put the following line into the VM options in the run/debug configurations window to enable the net debugging. 

```
-Djavax.net.debug=all
```

[keystore and truststore]: /tutorial/security/keystore-truststore/
[keytool]: /tool/keytool/

