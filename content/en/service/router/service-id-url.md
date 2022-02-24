---
title: "Service Id Url"
date: 2022-02-24T17:04:02-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When using light-router to route traffic to the downstream APIs, most users will use PathPrefixServiceHandler, PathServiceHandler or ServiceDictHandler to derive the downstream API serviceId from the request attributes like the path, method or endpoint etc. 

The above setup works great if the downstream APIs are different with different paths and endpoints. What if we have the same API deployed with multiple instances to serve clients from different geo-locations? How can we route the traffic to different hosts when the requests are similar?

In this case, we have to pass a service_id or service_url in the HTTP header to assist the light-router in routeing the requests to the right host. 

Use service_url if your downstream API has a domain name with a static IP address to bypass the service discovery and go to the host directly in the router. 

Use service_id in the header to tell the router to use this service_id to discover the target service hosts. Once a list of hosts is available, the router load balances to one. It is suitable when our services are deployed to the cloud without a static IP address. 

Most client applications can manipulate HTTP headers to add the service_id or service_url; however, some client applications cannot manipulate headers. We have added a new flag in the router.yml to overwrite the service_id in the query parameters to support those clients. 


```
# support serviceId in the query parameter for routing to overwrite serviceId in header routing.
# by default, it is false and shouldn't be used unless you are dealing with a legacy client that
# doesn't support header manipulation. Once this flag is true, we are going to overwrite the header
# service_id derived with other handlers from prefix, path, endpoint etc.
serviceIdQueryParameter: ${router.serviceIdQueryParameter:false}

```

After you set the above config property to true, you can have a query parameter with key as service_id in the URL to indicate which service is about to invoke. 

After you set the above config property to true, you can have a query parameter with key as service_id in the URL to indicate which service is about to invoke. The service_url in the query parameter is not supported due to the URL encoding conflict. 


For example, you can have the following URLs to invoke different services in the US Kubernetes and Canadian Kubernetes clusters.

```
https://example.com/kyc/address?service_id=com.networknt.address.us
https://example.com/kyc/address?service_id=com.networknt.address.ca
```

