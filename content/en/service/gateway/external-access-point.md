---
title: "External Access Point"
date: 2023-07-14T15:44:57-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The light gateway can be used as a full-blown API gateway, or it can be used as a micro-gateway. One of the use cases is to wrap one or more internal  APIs and expose them as a single API to third-party consumer applications on the Internet or Intranet. This is called an external access point. 

By using the light gateway, we can hide the detailed implementation of the APIs and give third-party consumers a stable and unified API specification. Internally, we can expose only some endpoints from several APIs to third-party users. The access endpoint can enforce the security and other cross-cutting concerns before invoking internal APIs. 

Some organizations that want to re-branding third-party APIs can use this external access point to bridge the access from third-party consumers and third-party APIs. From the consumer perspective, they are accessing an internal API from your organization with your OAuth 2.0 credentials. In fact, they are just accessing an API in the cloud or from one of your partners. 

Instead of implementing a customized wrapper API, you can use the light gateway with external access point configuration for your use case. 


Here is a video tutorial. 

{{< youtube h0fGpd-u6D0 >}}
