---
title: "Client Tutorial"
date: 2018-06-06T23:03:56-04:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

To improve the performance and reduce the complexity of the HTTP connection pooling, we have developed [Http2Client][] which is HTTP/2 enabled for the client to service and the service to service communication. We also support one-way SSL by default for all communication channel and two-way SSL optional with the configuration in the server.yml file.

In most service to service interactions, we are using self-signed certificates which are managed by server.keystore and client.truststore. For more details on how to use these keystore files in the config folder, please refer to [keystore vs. truststore][]. Please beware that you also need a client.keystore and a server.truststore if two-way SSL is used. 

When communicating with public websites or API servers, it would be great to get the certificate from the owner. If we cannot, the following is a [Public HTTPS][] tutorial to download the certificate from a public website. 

- [Access Public HTTPS Site](/tutorial/client/public-https/)
- [Standalone Consul Discovery](/tutorial/client/consul-discovery/)
- [Client Dependencies](/tutorial/client/dependencies/)
- [Client Configuration](/tutorial/client/configuration/)
- [Client Usage](/tutorial/client/code/)
- [Get JWT Token](/tutorial/client/jwt-token/)
- [Multiple JWT Tokens](/tutorial/client/multiple-tokens/)
- [Get JWK](/tutorial/client/get-jwk/)
- [Multiple JWKs](/tutorial/client/multiple-jwks/)


[keystore vs. truststore]: /tutorial/security/keystore-truststore/
[Public HTTPS]: /tutorial/client/public-https/
[Http2Client]: /concern/client/