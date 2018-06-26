---
title: Client Module Code
linktitle: Client Module Code 
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: "Java API for interacting with the client module within your codebase."
weight: 30
sections_weight: 30
draft: false
toc: true
---

## Introduction

Usage of the client module can support a wide variety of integration patterns to the destination
service.


## Obtaining the URL.

The first thing required is to get the location of the api you wish to call. This is done
using the `Cluster` interface, and passing a few parameters to a call to `serviceToUrl`.

```java
static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
// protocol, serviceId, environment, requestKey (load balancer key, not required in most cases)
String apiHost = cluster.serviceToUrl("https", "com.networknt.apib-1.0.0", tag, null);
```

Based on the location that the `com.networknt.apib-1.0.0` service registers itself in the
instance of consul pointed to from `service.yml`, it will return an `https` url to that
one of the instances.

## Calling the Service

Given a url, first open a connection to that server:

```java
static Http2Client client = Http2Client.getInstance();
ClientConnection connection = client.connect(new URI(apiHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
```

If the service fails to connect, an exception will be thrown.
The next thing we want to do is get a reference to the response, build our latches for asynchronous processing, and generate a request.


```java

// Get a reference to the response
final AtomicReference<ClientResponse> reference = new AtomicReference<>();
// We only expect one response, so close the latch after it's received.
final CountDownLatch latch = new CountDownLatch(1);

// Build the request object:
ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(apiPath);
```

After we have these objects created, we now only have to send the request, and wait for the response.

```java

// Send the request
connection.sendRequest(request, client.createClientCallback(reference, latch));

// Wait for the response, SLA is 100 milliseconds for example.
latch.await(100, TimeUnit.MILLISECONDS);


// Read the response.
ClientResponse response = reference.get();
```

Closing the connection, if necessary could be done

## FAQ

How would I get the HTTP response code of the request:

```java
int statusCode = response.getResponseCode();
``` 

How would I get the body (content) of the response:

```java
String responseBody = response.getAttachment(Http2Client.RESPONSE_BODY)
```

If that `responseBody` was in json, could I convert it to an object?

```java
List<String> responseList = Config.getInstance().getMapper().readValue(responseBody, new TypeReference<List<String>>(){});
```