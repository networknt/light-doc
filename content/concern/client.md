---
title: "Client"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

## Introduction

In microservices architecture, service to service communication can be done by request/response style or messaging/event style. An efficient HTTP client is crucial in request/response style as the number of interactions between services are high and extra latency can kill the entire application performance to cause the failure of a microservices application.

In the early days of light-4j, we have a client module based on Apache HttpClient and Apache HttpAsyncClient which supports HTTP 1.1 and is very popular in the open source community. However, it was designed long ago, and it is tough to use because of too many configurations. It is also huge, and slow compare with other modern HTTP clients. Another big issue is HTTP 2.0 support as light-*-4j frameworks support HTTP2 natively and we want to take advantage of the client side as well.

In looking for Java HTTP clients that support HTTP 2.0, we were stuck there as none of them supports it gracefully. Some are partial support, and most of them require you to put -Xbootclasspath with a specific version of jar file per JDK version in the command line to work with Java 8. Everybody seems to wait for Java 9 to come out, but we cannot use it on production until it is ready.

Given the above situations, I decided to implement our own Http2Client based on what we have in Undertow. I have proposed the idea to implement a generic Http2Client to Undertow community, but it wasn't interested. It will take a long time to build an independent Http2Client without depending on Undertow, and it is OK with us as our server is based on Undertow anyway. Other people might have a concern on that, but the argument is Undertow core is extremely small, and I don't think it is an issue for other people to use it outside of light-*-4j frameworks.

I am starting an [http client benchmark](https://github.com/networknt/http2client-benchmark)
and if there are more interests on this client, I will make it an independent module without depending on Undertow core so that other people working on other platforms can use it without the extra Undertow core.

## Usage

Http2Client supports both HTTP 1.1 and HTTP 2.0 transparently depending on if the target server supports HTTP 2.0 and is used to call APIs from the following sources:

1. Web Server
2. Standalone Application/Mobile Application
3. API/Service

It provides methods to get authorization token and automatically gets client credentials token for scopes in API to API calls. It also helps to pass correlationId and traceabilityId and other configurable headers to the next service.

Although it supports HTTP 1.1, it requires the user to create a connection pool as HTTP 1.1 connection doesn't support multiplex and one connection can only handle one request concurrently. I am planning to add an internal connection pool if there is a need but currently it is not necessary as all the communication is on HTTP 2.0

Http2Client is a very low-level component, and it is best to be used in service to service communication. If you are trying to write an original client application in Java 8, please take a look at [light-consumer-4j][] which is written by Nicholas Azar and contributed by the community. It is built on top of Http2Client and has a lot of extra features like connection pooling etc. 

### Localhost vs 127.0.0.1

When using the client module to call a service on the same host, you can use localhost or 127.0.0.1 before release 1.5.29 as there was a hostname verification bug in the framework. Since 1.5.29, you must use localhost in the URL as the bug was fixed and localhost is matching the hostname in the generated client.truststore and server.keystore files. 

### Generic response callback functions

Like Undertow core server, it is event-driven with callbacks so non-blocking all the time to free your CPU for other important computation. Http2Client has two generic callback functions implemented to handle requests with a body (POST, PUT, PATCH) and requests without a body(GET, DELETE). Users can create their own customized callback functions to handle the response from the server if they need special logic inside the callback function.

Signature for GET/DELETE

```
public ClientCallback<ClientExchange> createClientCallback(final AtomicReference<ClientResponse> reference, final CountDownLatch latch)
```

Signature for POST, PUT, PATCH

```
public ClientCallback<ClientExchange> createClientCallback(final AtomicReference<ClientResponse> reference, final CountDownLatch latch, final String requestBody)
```

Note that you need to pass in a requestBody for POST, PUT or PATCH.

### Client Credentials token renew and cache

The renew of token happens behind the scene, and it supports the circuit breaker if the OAuth 2.0 server is down or busy. It renews the token pro-actively before the current one is expired and lets all requests go with the current token. It only blocks other requests if the current request is trying to renew an expired token. When token renewal in this case fails, all requests will be rejected with a timeout and subsequent requests the same until a grace period is passed so that the renew process starts again. 

There is a good reason we renew the token proactively. If we leave the token to the expiration, the traditional API service can return 401 error with the token expired message to notify the client to get a new token. In the microservices architecture, this is not possible. What if the token is one second before the expiration and service A accepts it and write something into its database and then service B accepts it and write something into its database. However, when service C receives it, it is expired already and rejected. There must be extra logic to compensate the transactions service A and B have performed already. We have a framework [light-saga-4j][] for this type of microservices orchestration, but it is really not necessary for just token expiration handling. It would be better to handle it gracefully in the client module to ensure that the token sent has a longer expiration time than the entire application SLA. 


### Examples

#### Get a client instance

Http2Client is a singleton, and you only need one instance in your application. You can create as many connections from the same instance though. 

```
    Http2Client client = Http2Client.getInstance();
```

#### Call from web server

To set the header with authorization code JWT token.

```
    public void addAuthToken(HttpRequest request, String token) 
```

To set the header with authorization code JWT token and traceabilityId.

```
    public void addAuthTokenTrace(HttpRequest request, String token, String traceabilityId) 
```

#### Call from standalone app/mobile app

To set the header with client credentials JWT token.
```
    public void addCcToken(HttpRequest request) throws ClientException, ApiException 
```

To set the header with client credentials JWT token and traceabilityId.

```
    public void addCcTokenTrace(HttpRequest request, String traceabilityId) throws ClientException, ApiException 

```

#### call from API/service

To pass in exchange.

```
    public void propagateHeaders(HttpRequest request, final HttpServerExchange exchange) throws ClientException, ApiException 

```

To pass in variables individually.

```
    public void populateHeader(HttpRequest request, String authToken, String correlationId, String traceabilityId) throws ClientException, ApiException 

```

### Test cases that use Http2Client

All tests in light-*-4j frameworks are real tests against the real server, and the test cases are using Http2Client to communicate with the server. We have several hundred test cases in all different frameworks, and light-oauth2 concentrates some of them. Take a look at https://github.com/networknt/light-oauth2/tree/master/client/src/test/java/com/networknt/oauth/client/handler for examples on how to write test cases with Http2Client. In fact, you can check other modules in light-oauth2 for test case examples as well.

### Http2Client communicate with third party server.

Within light-4j, there are several places that Http2Client is used to communicate with third party servers

* OAuth2 server

https://github.com/networknt/light-4j/blob/master/client/src/main/java/com/networknt/client/oauth/OauthHelper.java

* Consul

https://github.com/networknt/light-4j/blob/master/consul/src/main/java/com/networknt/consul/client/ConsulClientImpl.java

* InfluxDB

https://github.com/networknt/light-4j/blob/master/metrics/src/main/java/io/dropwizard/metrics/influxdb/InfluxDbHttpSender.java

* Client folder in light-example-4j

https://github.com/networknt/light-example-4j/tree/master/client

* post-client

https://github.com/networknt/light-example-4j/tree/master/router


### Tutorials that use Http2Client

* [Microservices Chain pattern tutorial][]

* [Service discovery tutorial][]


### Close connection

In most of the cases, we don't close the connection, and all the requests will go through the same HTTP/2 connection with multiplex support. However, there are times you need to create a new connection on demand and close it once it is used. For example, in [Server.java](https://github.com/networknt/light-4j/blob/master/server/src/main/java/com/networknt/server/Server.java) we create a new connection to load config from light-config-server during server startup. In this case, although our config server supports HTTP/2, we create and close the connection as this is a one-time request and we don't want to hold the resource. Here are the lines to close the connection in finally block.

```java
    } finally {
        IoUtils.safeClose(connection);
    }
```

Although it is recommended to keep the connection cached for high volume services, the connection must be closed from time to time so that the load balancer can redirect the same client instance to different server instance unless you want to pin a particular client instance to a server instance for a long time. This will eventually cause imbalance load on the service instances if there are not too many different clients. 

The Consul client is one of the examples that the connection is reset from time to time to ensure that we are not running out of max number of requests per connection in HTTP/2. The current connection will be dropped and recreated after 1 million requests. 

https://github.com/networknt/light-4j/blob/master/consul/src/main/java/com/networknt/consul/client/ConsulClientImpl.java

The easier way to close collection periodically is to close it after a number of requests or close it after a number of minutes. The same finally block can be written as.

```java
    } finally {
        if(number of minutes passed or number of requests handled) {
            IoUtils.safeClose(connection);
        }
    }

```

### Check connection

When you cache the connection for a while, you need to ensure that the connection is still open as it might be closed from the server or closed periodically to re-balance the load on server instances. 

Here is the example to check the connection before using it.

```java
        if(connectionC == null || !connectionC.isOpen()) {
            try {
                apicHost = cluster.serviceToUrl("https", "com.networknt.apic-1.0.0", null);
                connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
```

The above code checks the cached connectionC and if it is closed already, then create a new connection and cache it again.

For the full example, please refer to https://github.com/networknt/light-example-4j/blob/master/discovery/api_a/consul/src/main/java/com/networknt/apia/handler/DataGetHandler.java

### Multiple Request Pattern

If you send multiple request through one connection, here is an example. 

```
    public void testRouterHttps48k() throws Exception {
        ClientConnection connection = null;
        try {
            connection = client.connect(new URI("https://localhost:8000"), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            final String json = getResourceFileAsString("48k.json");
            for(int i = 0; i < 100; i++) {
                final CountDownLatch latch = new CountDownLatch(1);
                final AtomicReference<ClientResponse> reference = new AtomicReference<>();
                final ClientRequest request = new ClientRequest().setMethod(Methods.POST).setPath("/v1/postData");
                request.getRequestHeaders().put(Headers.HOST, "localhost");
                request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
                request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
                connection.sendRequest(request, client.createClientCallback(reference, latch, json));
                latch.await(1000, TimeUnit.MILLISECONDS);
                int statusCode = reference.get().getResponseCode();
                String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
                if(statusCode != 200) System.out.println("testRouter4k failed with statusCode = " + statusCode + " body = " + body);
                System.out.println(i);
                if(!body.startsWith("[")) System.out.println(body);
            }
        } finally {
            if(connection != null) IoUtils.safeClose(connection);
        }
    }

```
### Get Request

Setting GET parameters via URL string.

```
ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath("/oauth2/client?page=1&api_key=asdfsomthingelse");
```

You might need to set some headers. 

```
request.getRequestHeaders().put(Headers.HOST, "localhost");
request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
```

### Post Request

I got a lot of questions on how to make a post request to the server and above is an example. Please remember the following. 

* You need the following headers. 

```
  request.getRequestHeaders().put(Headers.HOST, "localhost");
  request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
  request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
```

* You need to pass the request body into the callback funcation. 

```
connection.sendRequest(request, client.createClientCallback(reference, latch, json));
```

### NPE at reference.get()

When accessing a server with heavy load, the response time might be longer. In the above example, we waited 1000ms to ensure that the response is comming back. If we only wait 10ms, chances are the response is not backed yet and the subsequent call to get statusCode will throw NullPointerException. If you see this exception, try to increase the await timeout to bigger number like 1000ms. In the above example, we are sending 100 big requests(48k) to the server so the wait time is set as 1000ms. 

```
latch.await(1000, TimeUnit.MILLISECONDS);
```

If you want to wait until the default timeout defined in client.yml is reached, you can just call. 

```
latch.await();
```

For some slow services, you might need to adjust the default timeout in client.yml to allow the client to wait longer before timeout. This is usually dealing with legacy services, and these services should be upgraded with the async approach if possible. 

### HOST Header

When using Http2Client, you need to add the following header to the request. 

```
request.getRequestHeaders().put(Headers.HOST, "localhost");
```

You can see the entire example in the above testRouterHttps48k method. 


### Configuration

When using client module of light-4j, you need to have a configuration file client.yml in your src/main/resources/config folder. If OAuth 2.0 is used to secure the API access, then you need to update secret.yml to have client secrets configured there.

The following is the default client.yml in light-4j and you should replace it with an externalized client.yml file.

```yaml
# This is the configuration file for Http2Client.
---
# Buffer Size in the buffer pool in KB. If should be bigger than your request or response body size.
bufferSize: 24
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: client.keystore
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://localhost:8080/authorization_code
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    refresh_token:
      # token endpoint for refresh token grant
      uri: "/oauth2/token"
      # client_id for refresh token grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
  # light-oauth2 key distribution endpoint configuration
  key:
    # key distribution server url
    server_url: https://localhost:6886
    # the unique service id for key distribution service
    serviceId: com.networknt.oauth2-key-1.0.0
    # the path for the key distribution endpoint
    uri: "/oauth2/key"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
  # de-ref by reference token to JWT token. It is separate service as it might be the external OAuth 2.0 provider.
  deref:
    # Token service server url, this might be different than the above token server url.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider. Need for service lookup/discovery.
    serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true
    # the path for the key distribution endpoint
    uri: "/oauth2/deref"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72
``` 

Please be aware that bufferSize need to be increased to the size of your maximum request body. The default 24*1024 should be good enough for most of the application. There is a [tutorial][] to give you more details on the bufferSize configuration. 


And here is an example of secret.yaml with client secrets.

```yaml
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Fresh token client secret for OAuth2 server
refreshTokenClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# De-Reference access token to JWT token client secret.
derefClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: token1

# EmailSender password default address is noreply@lightapi.net
emailPassword: change-to-real-password

```


[Microservices Chain pattern tutorial]: /tutorial/rest/swagger/ms-chain/
[Service discovery tutorial]: /tutorial/common/discovery/
[tutorial]: https://github.com/networknt/light-example-4j/tree/master/router
[light-consumer-4j]: https://github.com/networknt/light-consumer-4j
[light-saga-4j]: /style/light-saga-4j/
