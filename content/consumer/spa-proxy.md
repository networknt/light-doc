---
title: "Single Page App Proxy"
date: 2018-05-13T23:52:10-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When developing React single page application with a client-side framework like React, it is wise to take advantage of WebPack hot loader for higher productivity. This requires that we start a Nodejs server to serve the single page application to the browser. When the single page application is calling some of the APIs, the API servers might be on another host or the same host but with a different port number. To make sure that Javascript application only needs to specify the relative path without worry about the host and port, we need to setup a proxy in the generated package.json file. 

The following is an example we are using for development. 

```
proxy: {
  "/v1": {
    target: "https://localhost:8443",
    secure: false
  }
}
```

Above configuration will ensure that all request to /v1 path will be routed to https://localhost:8443 service. As the target server is listening to HTTPS, we need to set the secure to false if a self-signed certificate is used on the server. 

