---
title: "Path Prefix Service"
date: 2018-07-30T12:34:21-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The original [PathServiceHandler][] has hard dependency on Swagger or OpenAPI handlers in the light-rest-4j framework. It is feature rich, but sometimes overkill for simple use cases. For a simple mapping from a request path to a serviceId, a pattern matching will be good enough. @logi contributed this handler and it is very good if you want to replace the [PathServiceHandler][]. 

When using the light-router, each request must have a serviceId in the header in order to allow the router to discover the services before invoking downstream services. The reason we have to do that is due to the unpredictable path between services. If you are sure that a unique path prefix can identify all the downstream services, then you can use this PathPrefix to ServiceId mapper handler to uniquely identify the serviceId and put it into the header. In this case, the client can invoke the service just the same way as it is invoking the service directly.

Please note that you cannot invoke /health or /server/info endpoints as these are the common endpoints injected by the framework and all services will have them on the same path. The router cannot figure out which service you want to invoke so an error message will be returned. 

Unlike [PathServiceHandler][], this handler does not require OpenAPIHandler or SwaggerHandler but is also unable to do any validation beyond the path prefix. 

For information on the configuration, please refer to [router configuration][]. 

[PathServiceHandler]: /service/router/path-service/
[router configuration]: /service/router/configuration/
