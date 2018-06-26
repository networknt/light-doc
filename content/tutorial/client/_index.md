---
title: "Client Tutorial"
date: 2018-06-06T23:03:56-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 10
aliases: []
toc: false
draft: false
---

To improve the performance and reduce the complexity of the HTTP connection pooling, we have developed Http2Client which is HTTP/2 enabled for client to service and service to service communication. We also support one-way SSL by default for all communication channel and two-way SSL optional with the configuration in server.yml file.

In most service to service interactions, we are using self-signed certificates which are managed by server.keystore and client.truststore. For more details on how to use these keystore files in config/tls folder, please refer to [keystore vs truststore][]

When communicating with public websites or API servers, it would be great to get the certificate from the owner. If we cannot, the following is a tutorial to download the certificate from a public website. 

[keystore vs truststore]: /tutorial/security/keystore-truststore/
[Public HTTPS]: /tutorial/client/public-https/
