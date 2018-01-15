---
title: "Service Registration"
date: 2017-11-10T14:50:30-05:00
description: ""
categories: []
keywords: []
weight: 50
aliases: []
toc: false
draft: false
---


OAuth2 is used to protect services and each service must register itself with endpoints and 
scopes in order to have fine-grained access control. This microservice provides endpoint to 
add a new service, update an existing service, remove an existing service and query service(s).

It also all you to optionally register service endpoints with scopes so that it can help with
client registration and scope calculation.  

In this tutorial, we are using curl command to demo how these services are working. In
reality, these endpoints are accessed from light-portal UI.


To add a new service.

```
curl -k -H "Content-Type: application/json" -X POST -d '{"serviceId":"AACT0003","serviceType":"openapi","serviceName":"Retail Account","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","ownerId":"admin"}' https://localhost:6883/oauth2/service
```

To query all services.

```
curl -k https://localhost:6883/oauth2/service?page=1

```
And here is the result.

```
[{"serviceType":"openapi","serviceDesc":"A microservice that serves account information","scope":"a.r b.r","serviceId":"AACT0001","serviceName":"Account Service","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"},{"serviceType":"ms","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"}]
```

To query a service with service id.

```
curl -k https://localhost:6883/oauth2/service/AACT0003

```
And here is the result.
```
{"serviceType":"openapi","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin"}
```

To update above service type to "api".

```
curl -k -H "Content-Type: application/json" -X PUT -d '{"serviceType":"swagger","serviceDesc":"Microservices for Retail Account","scope":"act.r act.w","serviceId":"AACT0003","serviceName":"Retail Account","ownerId":"admin"}' https://localhost:6883/oauth2/service
```

To delete above service with service id.

```
curl -k -X DELETE https://localhost:6883/oauth2/service/AACT0003

```

To add several endpoints along with scopes to a service. 


```
curl -k -H "Content-Type: application/json" -X POST -d '[{"endpoint":"/v1/data@post","operation":"createData","scope":"data.w"},{"endpoint":"/v1/data@put","operation":"updateData","scope":"data.w"},{"endpoint":"/v1/data@get","operation":"retrieveData","scope":"data.r"},{"endpoint":"/v1/data@delete","operation":"deleteData","scope":"data.w"}]' https://localhost:6883/oauth2/service/AACT0001/endpoint
```