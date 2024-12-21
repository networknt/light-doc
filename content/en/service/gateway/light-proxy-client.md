---
title: "Light Proxy Client"
date: 2022-11-29T18:36:00-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Light Proxy Client (LPC) runs as a separate light-gateway service on the same host as API consumer(s). It allows the legacy consumer applications to be integrated into the light-4j ecosystem without changing a line of code. Also, it will be responsible for getting the OAuth 2.0 token from the authorization server and adding the token to the authorization header in the request to the downstream API. 

### Prerequisites

* Java 11 installed.
* The authorization server credentials.

### Client Library


