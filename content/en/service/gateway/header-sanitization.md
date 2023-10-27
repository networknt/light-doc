---
title: "Header Sanitization"
date: 2023-10-27T10:31:24-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

As part of the ecosystem, the light gateway must pass the X-Correlation-Id and X-Traceability-Id to the downstream APIs. To prevent XSS attacks, we normally set up the sanitizer handler on the gateway to encode the above two headers. Of course, you can do the same for other headers based on your business requirements. 

### Handler

The first step is to add the SanitizerHandler to the handler registration and the default chain in the values.yml file. 

```
handler.handlers:
  .
  .
  .
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  .
  .
  .

handler.chains.default:
  .
  .
  .
  - header
  - sanitizer
  .
  .
  .
```

The position of the sanitizer should be after the header and before the router and proxy. 


### Config

With the sanitizer wired in the default chain, we need to configure it to encode the two headers mentioned above. 


```
sanitizer.enabled: true
sanitizer.bodyEnabled: false
sanitizer.headerAttributesToEncode: ["X-Correlation-Id","X-Traceability-Id"]
```

The above configuration will enable the sanitizer, disable the body sanitization, and enable the header sanitization with 'javascript-source' encoder for the two headers listed in JSON array format. 

