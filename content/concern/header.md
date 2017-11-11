---
title: "Header Handler"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 55
weight: 55	#rem
aliases: []
toc: false
draft: false
---

When a request pass through a list of middleware handlers, chances are you want to update
the request header or update response header as part of the cross-cutting concerns. The
HeaderHandler is designed for this. The very first use case is for light-proxy to update
the Authorization header from Bearer token to Basic Authorization when connecting to the
down stream services. This allows light-proxy to verify the OAuth 2.0 JWT access token on
the proxy and then change the same header to Basic so that the downstream services can be
called from light-proxy. 

The HeaderHandler manipulate both request header and response header based on a header.yml
configuration file. 

Here is an example of header.yml

```yaml
# Enable header handler or not, default to false
enabled: false
# Request header manipulation
request:
  # Remove all the headers listed here
  remove:
  - header1
  - header2
  # Add or update the header with key/value pairs
  # Although HTTP header supports multiple values per key, it is not supported here.
  update:
    key1: value1
    key2: value2

# Response header manipulation
response:
  # Remove all the headers listed here
  remove:
  - header1
  - header2
  # Add or update the header with key/value pairs
  # Although HTTP header supports multiple values per key, it is not supported here.
  update:
    key1: value1
    key2: value2
```

The config file is self-explained. In order to use it, there are two step in setup.

## Add HeaderHandler to com.networknt.handler.MiddlewareHandler config file

Here is an [example][] in light-proxy.

## Enable it in header.yml

The second step is to enable it in header.yml and put the proper properties in the
config file to add/update request/response headers or remove request/response headers. 


[example]: https://github.com/networknt/light-proxy/blob/master/src/main/resources/META-INF/services/com.networknt.handler.MiddlewareHandler
