---
title: "Service"
date: 2017-11-10T14:50:30-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


OAuth2 is used to protect services and each service must register itself with scope in
order to have fine-grained access control. This microservice provides endpoint to add a
new service, update an existing service, remove an existing service and query service(s). 

In this tutorial, we are using curl command to demo how these services are working. In
reality, these endpoints are accessed from light-portal UI.


To add a new service.

```
curl -k -H "Content-Type: application/json" -X POST -d '{"serviceId":"AACT0003","serviceType":"ms","serviceName":"Retail Account","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","ownerId":"admin"}' https://localhost:6883/oauth2/service
```

To query all services.

```
curl -k https://localhost:6883/oauth2/service?page=1

```
And here is the result.

```
[{"serviceType":"ms","serviceDesc":"A microservice that serves account information","scope":"a.r b.r","serviceId":"AACT0001","serviceName":"Account Service","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"},{"serviceType":"ms","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"}]
```

To query a service with service id.

```
curl -k https://localhost:6883/oauth2/service/AACT0003

```
And here is the result.
```
{"serviceType":"ms","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin"}
```

To update above service type to "api".

```
curl -k -H "Content-Type: application/json" -X PUT -d '{"serviceType":"api","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin"}' https://localhost:6883/oauth2/service
```

To delete above service with service id.

```
curl -k -X DELETE https://localhost:6883/oauth2/service/AACT0003

```
