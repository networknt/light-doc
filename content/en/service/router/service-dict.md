---
title: "Service Dict Mapping"
date: 2021-11-07T23:46:22-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When using the light-router or http-sidecar, each request must have service_id in the header in order to allow the router to perform the service discovery before invoking a downstream service. The reason we have to do that is due to the unpredictable path conflicts between services. If you are sure that the path or the endpoint can uniquely identify all the downstream services, then you can use this Path to ServiceId mapping handler to uniquely identify the serviceId and put it into the header. In this case, the client can invoke the service just the same way as it is invoking the service directly with fixed IP address and port number.

Please note that you cannot invoke /health or /server/info endpoints as these are the common endpoints injected by the framework and all services will have them on the same path. The router cannot figure out which service you want to invoke so an error message will be returned.

This handler depends on the request path and HTTP method. In this handler, we are using the retrieved request path and method combination to look up a unique service from the configuration mapping populated from the serviceDict.yml config file. 

Suppose the validator handler is deployed on the router along with the combined specification from downstream services. In that case, invalid requests can be blocked from the light-router or the http-sidecar instead of hitting the backend services. 

In case you don't want to validate the request on the router or it is very hard to construct a combined specification to enforce validation, you can replace this handler with [PathPrefixServiceHandler][] which is contributed from @logi, and it is much simpler to use. 

For information about the config file, please refer to [router configuration][].

[router configuration]: /service/router/configuration/
[PathPrefixServiceHandler]: /service/router/path-prefix-service/
