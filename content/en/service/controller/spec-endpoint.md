---
title: "Specification Endpoint"
date: 2021-10-24T23:16:36-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The OpenAPI 3.0 specification can be found in the resources/config folder in the light-controller [repository](https://github.com/networknt/light-controller/blob/master/src/main/resources/config/openapi.yaml). 


### Get All Services


```
paths:
  '/services':
    get:
      summary: Query all healthy services
      description: Returns a list of services
      operationId: getHealthService
      responses:
        '200':
          description: successful operation
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Status'
        '404':
          description: Service not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Status'
      security:
        - portal_auth:
            - 'A8E73740C0041C03D67C3A951AA1D7533C8F9F2FB57D7BA107210B9BC9E06DA2'
```

In the cluster mode, this endpoint gets all the live services across the cluster, and it also filters out any stale services with health check tasks not being generated from the light-scheduler. The situation only happens when all nodes of the light-scheduler are down simultaneously, or messages are lost in the light-scheduler topic on Kafka. 

When the health check task is missing, no health check probe will be issued from the health check streams, and there will be no update to the cluster-wide health check store for the service. It means there will be no proper way to remove the stale service from the store. To ensure the accuracy of the service registry, we need to find a way to remove those services if health check tasks are not scheduled. 

The ServicesGetHandler will be called in two different modes: local vs cluster. 

In local mode, it will only return all the services registered to the local store, and it is usually called by the main node that is invoked by the UI to get cluster-wide services. 

When the local query parameter is false or missing, the handler will get all the local services and call other nodes in the cluster to get their services. Before returning the result, it will call the following statement to get the stale health checks. 

```
Map<String, Object> checks = ServicesCheckGetHandler.getClusterHealthChecks(exchange, true);

```
The above method will check the stale health checks from all nodes, send events to de-register the service, and stop the health check scheduler if the health check status is stale. We identify stale health checks if the last execution timestamp is older than the deregisterCriticalServicesAfter(2 minutes by default). 

Since the update of the service registry and health check stores are async,   we need to use the stale health checks to filter out the final services in the response. It is done by the following method to filter the services with checks. 

```
filterServiceByCheck(services, checks)
```

It will ensure the final result list only contains the live services and their instances; even some of the events are just sent to remove the stale services from the store through Kafka messages. 

When we filter the instances from service by the stale health checks, the service will be removed if the last instance is removed from the list. 


### Lookup Service

```
  '/services/lookup':
    get:
      summary: Discover service with serviceId and optional tag
      description: Returns a list of nodes for the service
      operationId: getLookupService
      parameters:
        - name: serviceId
          in: query
          description: ID of the service
          required: true
          schema:
            type: string
        - name: tag
          in: query
          description: Return only nodes with the tag
          required: false
          schema:
            type: string
      responses:
        '200':
          description: successful operation
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Status'
        '404':
          description: Service not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Status'
      security:
        - portal_auth:
            - 'A8E73740C0041C03D67C3A951AA1D7533C8F9F2FB57D7BA107210B9BC9E06DA2'


```

The endpoint is designed for client-side service discovery. Any API can invoke this endpoint with a serviceId and an optional tag to find a list of instances of downstream API. 

We also take the opportunity to clean up stale services with stale health checks. Since this endpoint will be invoked frequently, we want to cached stale checks for 60 seconds to reduce the query to /services/check endpoint. 

Also, the lookup result will be filtered by the stale checks. 





