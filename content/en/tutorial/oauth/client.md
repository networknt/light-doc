---
title: "Client Registration"
date: 2017-11-10T14:50:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Every client that accesses service(s) must register itself in order to get the
access token during runtime from OAuth 2.0 provider. If it happens that the client is
an service and some other client is calling it. This particular API will need to register
itself twice. One as a client and one as a service. 

The client service provides an endpoint to create a new client, update an existing client and
delete a client. Here we have the services accessed by curl command for demo purpose. In
reality, these APIs will be accessed from light-portal UI. 


To add a new client.

```
curl -k -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"mobile","clientName":"AccountViewer","clientDesc":"Retail Online Banking Account Viewer","scope":"act.r act.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' https://localhost:6884/oauth2/client
```

And here is the result with client_id and client_secret.

```
{"clientDesc":"Retail Online Banking Account Viewer","clientType":"public","clientProfile":"mobile","redirectUri":"http://localhost:8080/authorization","clientId":"e24e7110-c39f-49f1-85eb-8434cb577482","clientName":"AccountViewer","scope":"act.r act.w","clientSecret":"YDJLse8SQRapHyoMsdPUig","ownerId":"admin","createDt":"2016-12-31"}
```

To query all clients.

```
curl -k https://localhost:6884/oauth2/client?page=0

```
And here is the result.

```
[{"clientDesc":"PetStore Web Server that calls PetStore API","clientId":"f7d42348-c647-4efb-a52d-4c5787421e72","clientType":"public","clientProfile":"mobile","redirectUri":"http://localhost:8080/authorization","clientName":"PetStore Web Server","scope":"petstore.r petstore.w","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"},{"clientDesc":"Retail Online Banking Account Viewer","clientId":"9ef89c7b-f17b-4a64-a24b-ce539ed80641","clientType":"public","clientProfile":"mobile","redirectUri":"http://localhost:8080/authorization","clientName":"AccountViewer","scope":"act.r act.w","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"}]
```

To query a client by id.

```
curl -k https://localhost:6884/oauth2/client/f7d42348-c647-4efb-a52d-4c5787421e72
```

And here is the result.

```
{"clientDesc":"PetStore Web Server that calls PetStore API","clientId":"f7d42348-c647-4efb-a52d-4c5787421e72","clientType":"public","clientProfile":"mobile","redirectUri":"http://localhost:8080/authorization","clientName":"PetStore Web Server","scope":"petstore.r petstore.w","clientSecret":"f6h1FTI8Q3-7UScPZDzfXA","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"}
```

To update a client with a shorter clientDesc.

```
curl -k -H "Content-Type: application/json" -X PUT -d '{"clientDesc":"PetStore Web Server","clientId":"f7d42348-c647-4efb-a52d-4c5787421e72","clientType":"public","clientProfile":"mobile","redirectUri":"http://localhost:8080/authorization","clientName":"PetStore Web Server","scope":"petstore.r petstore.w","clientSecret":"f6h1FTI8Q3-7UScPZDzfXA","ownerId":"admin","updateDt":null,"createDt":"2016-12-31"}' https://localhost:6884/oauth2/client
```

To delete a client with client id.

```
curl -k -X DELETE https://localhost:6884/oauth2/client/9ef89c7b-f17b-4a64-a24b-ce539ed80641

```

The following section will try to link the client to a service and its endpoints. 

First let's create endpoints for an existing service. Pay attention at the scopes for each enpoint. 

```
curl -k -H "Content-Type: application/json" -X POST -d '[{"endpoint":"/v1/data@post","operation":"createData","scope":"data.w"},{"endpoint":"/v1/data@put","operation":"updateData","scope":"data.w"},{"endpoint":"/v1/data@get","operation":"retrieveData","scope":"data.r"},{"endpoint":"/v1/data@delete","operation":"deleteData","scope":"data.w"}]' https://localhost:6883/oauth2/service/AACT0001/endpoint
```

Now let's link the above service endpoints to a client. 

```
curl -k -H "Content-Type: application/json" -X POST -d '["/v1/data@post","/v1/data@put","/v1/data@get","/v1/data@delete"]' https://localhost:6884/oauth2/client/f7d42348-c647-4efb-a52d-4c5787421e72/service/AACT0001
```

Here is the response with old scope and new scope after the change for the client.

```json
{"old_scope":"petstore.r petstore.w","new_scope":"data.w data.r"}
```

Let's query the client service endpoints with both clientId and serviceId. 

```
curl -k https://localhost:6884/oauth2/client/f7d42348-c647-4efb-a52d-4c5787421e72/service/AACT0001

```

And here is the result. 

```json
["/v1/data@delete","/v1/data@get","/v1/data@post","/v1/data@put"]
```

We can also query for the clientId only to list all services and their endpoints for a client.

```
curl -k https://localhost:6884/oauth2/client/f7d42348-c647-4efb-a52d-4c5787421e72/service
```

And here is the result. 

```json
{"AACT0001":["/v1/data@delete","/v1/data@get","/v1/data@post","/v1/data@put"]}
```

Now let's remove all endpoints by a clientId and a serviceId. You can also remove all services linked
to a client. 

```
curl -k -X DELETE https://localhost:6884/oauth2/client/f7d42348-c647-4efb-a52d-4c5787421e72/service/AACT0001

```

And here is the result. 

```json
{"old_scope":"data.w data.r","new_scope":""}
```
