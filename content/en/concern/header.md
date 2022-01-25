---
title: "Header Handler"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

When a request passes through a list of middleware handlers, chances are you want to update the request header or update the response header as part of the cross-cutting concerns. The HeaderHandler is designed for this. The very first use case is for light-proxy to update the Authorization header from Bearer token to Basic Authorization when connecting to downstream services. This allows light-proxy to verify the OAuth 2.0 JWT access token on the proxy and then change the same header to Basic so that the downstream services can be called from light-proxy.

### Configuration

The HeaderHandler manipulates both the request header and response header based on a header.yml configuration file.

Here is an example of header.yml without externalized template. 

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

Here is the header.yml in the module src/main/resources/config folder. This is the default cnnfig file for the module if externalized header.yml is not provided in your application. 

```yaml
# Enable header handler or not. The default to false and it can be enabled in the externalized
# values.yml file. It is mostly used in the http-sidecar, light-proxy or light-router.
enabled: ${header.enabled:false}
# Request header manipulation
request:
  # Remove all the request headers listed here. The value is a list of keys.
  remove: ${header.request.remove:}
  # Add or update the header with key/value pairs. The value is a map of key and value pairs.
  # Although HTTP header supports multiple values per key, it is not supported here.
  update: ${header.request.update:}
# Response header manipulation
response:
  # Remove all the response headers listed here. The value is a list of keys.
  remove: ${header.response.remove:}
  # Add or update the header with key/value pairs. The value is a map of key and value pairs.
  # Although HTTP header supports multiple values per key, it is not supported here.
  update: ${header.response.update:}

```

The following values.yml contains the properties to replace some variables in the above header.yml file. 

```yaml

header.enabled: true
header.request.remove:
  - header1
  - header2
header.request.update:
  key1: value1
  key2: value2

header.response.remove:
  - header1
  - header2
header.response.update:
  key1: value1
  key2: value2
```

The combined result is the same as the first expanded header.yml example. 

### Usage 

In order to use it, there are two steps in its setup.

* Add HeaderHandler to handler.yml request/response chain

* Enable it to add a file like header.yml example or values.yml to the application resources/config folder.


If you use the values.yml file, you might receive an error message if the configuration is not correct. 




