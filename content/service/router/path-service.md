---
title: "Path Service Mapping"
date: 2018-05-29T18:40:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When using the router, each request must have service_id in the header in order to allow the router to perform the service discovery before invoking downstream service. The reason we have to do that is due to the unpredictable path conflicts between services. If you are sure that all the downstream services can be uniquely identified by the path or the endpoint, then you can use this Path to ServiceId mapping handler to uniquely identify the serviceId and put it into the header. In this case, the client can invoke the service just the same way it is invoking the service directly.

Please note that you cannot invoke /health or /server/info endpoints as these are the common endpoints injected by the framework and all services will have them on the same path. The router cannot figure out which service you want to invoke so an error message will be returned.

This handler depends on OpenAPIHandler or SwaggerHandler in the light-rest-4j framework. That means this handler only works with RESTful APIs. In rest swagger-meta or openapi-meta, the endpoint of each request is saved into an auditInfo object which is attached to the exchange for auditing. In this handler, we are using the retrieved endpoint to identify the service_id by a lookup in the mapping populated from pathService.yml config file. 

For information about the config file, please refer to [router configuration][].

[router configuration]: /service/router/configuration/
