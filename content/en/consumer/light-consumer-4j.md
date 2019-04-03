---
title: "Light Consumer 4j"
date: 2018-12-05T13:04:18-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

# Light Consumer 4j

Light Consumer 4j is module which helps for consumers to easily integrate with light-4j apis.
It is built on top of Http2Client & java8,it has a lot of extra features like connection pooling etc.
it supports both direct URL and service discover (Consul) to call the apis.
### Usage
Add dependency in your project.

```xml
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>light-consumer-4j</artifactId>
    <version>1.5.26</version>
</dependency>
```
Here are the methods available in ClientBuilder.

 Future <ClientResponse> send()           // Send request.
 
 String getServiceUrl()                   // Get the resolved serviceUrl
 
 disableHttp2()                           // Disable Http2, by default it is enabled.
 
 setClientRequest(ClientRequest reg)      // set client request to httpClientRequest.
 
 setRequestBody(String requestBody)       // set requestBody to httpClientRequest.
 
 setHeaderValue(HttpString headerName,..) // set headerName and headerValue to httpClientRequest.
 
 setApiHost(String apiUrl)	              // set apiUrl into httpClientRequest to call api with direct url .
 
 setConnectionCacheTTLms(long tTLms)      // set connection cache time out.
 
 setRequestTimeout(TimeoutDef timeout)    // set request time out
 
 setConnectionRequestTimeout(TimeoutDef t)// set connection request timeout
 
 setAuthToken(String authToken)           // Set the auth token in the request
 
 getAuthToken()                           // get authToken 
 
 addCCToken()                             // add the access token in request header and also checks access token expiration. 
 
 setServiceDef(ServiceDef serviceDef) // set protocol,service id , environment  and requestKey to call api via consul

 setMaxReqCount(int maxReqCount)          // enable connection cache by request number
 
 
 #### Here are the code examples to call the api.
 
Call the api with direct URL:

Example 1:
```
Future<ClientResponse> clientRequest = new HttpClientBuilder()
                      
    //set direct url
   .setApiHost("https://localhost:8453")
   .setClientRequest(new ClientRequest().setPath("/v1/customers/1").setMethod(Methods.GET))
   .setLatch(new CountDownLatch(1))
   .disableHttp2()
   .addCCToken()
   .setConnectionRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestBody("")
   .setConnectionCacheTTLms(10000)
   .setMaxReqCount(5)
   .send();
ClientResponse clientResponse = clientRequest.get();

```
Example 2:

```
            public static final String CUSTOMERS_URL = "https://localhost:8453";
            private static final String CUSTOMERS_PATH = "/v1/customers/{id}";
            
            HttpClientBuilder clientBuilder = new HttpClientBuilder();
            
            ClientRequest clientRequest = new ClientRequest().setMethod(Methods.GET)
            .setPath(CUSTOMERS_PATH.replace("{id}", id));
            
            clientBuilder.setClientRequest(clientRequest);
            clientBuilder.setLatch(new CountDownLatch(1));
            if (!Server.config.isEnableHttp2()) {
              clientBuilder.disableHttp2();
            }
            clientBuilder.setApiHost(CUSTOMERS_URL);
            clientBuilder.setConnectionCacheTTLms(20000);
            clientBuilder.setMaxReqCount(5)
            clientBuilder.setConnectionRequestTimeout(new TimeoutDef(10, TimeUnit.SECONDS));
            Future<ClientResponse> clientBuilderRequest = clientBuilder.send();
            ClientResponse clientResponse = clientBuilderRequest.get();
```

Call the api via service discover (Consul):

Example 1:

```
Future<ClientResponse> clientRequest = new HttpClientBuilder()
                     
    //set protocol,service id , environment  and requestKey 
   .setServiceDef(new ServiceDef("https", "training.customers-1.00.00","training", null))
   .setClientRequest(new ClientRequest().setPath("/v1/customers/1").setMethod(Methods.GET))
   .setLatch(new CountDownLatch(1))
   .disableHttp2()
   .addCCToken()
   .setConnectionRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestBody("")
   .setConnectionCacheTTLms(10000)
   .setMaxReqCount(5)
   .send();
ClientResponse clientResponse = clientRequest.get();

```
Example 2:

```  
            public static final String CUSTOMERS_SERVICE = "training.customers-1.00.00";
            private static final String CUSTOMERS_PATH = "/v1/customers/{id}";
            
            HttpClientBuilder clientBuilder = new HttpClientBuilder();
            
            ClientRequest clientRequest = new ClientRequest().setMethod(Methods.GET)
            .setPath(CUSTOMERS_PATH.replace("{id}", id));
            
            clientBuilder.setClientRequest(clientRequest);
            clientBuilder.setLatch(new CountDownLatch(1));
            if (!Server.config.isEnableHttp2()) {
                clientBuilder.disableHttp2();
            }
            clientBuilder.setServiceDef(new ServiceDef("https", CUSTOMERS_SERVICE, "training", null));
            clientBuilder.setConnectionCacheTTLms(20000);
            clientBuilder.setMaxReqCount(5);
            clientBuilder.setConnectionRequestTimeout(new TimeoutDef(10, TimeUnit.SECONDS));
            Future<ClientResponse> clientBuilderRequest = clientBuilder.send();
            ClientResponse clientResponse = clientBuilderRequest.get();
```
    
    

