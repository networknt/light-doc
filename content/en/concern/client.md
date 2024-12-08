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

The service-to-service communication can be done through a request/response style or messaging/event style in a microservices architecture. An efficient HTTP client is crucial in a request/response style as the number of interactions between services is high, and extra latency can kill the entire application performance, causing the failure of a microservices application.

In the early days of light-4j, we had a client module based on Apache HttpClient and Apache HttpAsyncClient, which supported HTTP 1.1 and was very popular in the open-source community. However, it was designed long ago and was hard to use because of its numerous configurations. It was also huge and slow compared to other modern HTTP clients. Another big issue was the HTTP 2.0 support as light-4j frameworks support HTTP2 natively, and we want to take advantage of the client-side as well.

While looking for Java HTTP clients that supported HTTP 2.0, we were stuck as none could support it gracefully. Some provided partial support, while most of them required you to put -Xbootclasspath with a specific version of the jar file per JDK version in the command line for it to work with Java 8. Everybody seems to want to wait for Java 11 to come out, but we cannot use it on production until it is ready.

Given the above situation, I decided to implement our own Http2Client based on what we have in Undertow. I proposed the idea of implementing a generic HTTP Client to the Undertow community, but it wasn’t interested. It will take a long time to build an independent HTTP 2Client without depending on Undertow, which is OK with us as our server is based on Undertow anyway. Others may have concerns, but our argument is that the Undertow core is extremely small, and I don’t think it is an issue for other people to use it outside of light-4j frameworks.

I am starting an [http client benchmark](https://github.com/networknt/http2client-benchmark). If there are more users interested in this client, I will make it an independent module without depending on the Undertow core so that other people working on other platforms can use it without the extra Undertow core.


## Usage

Http2Client supports both HTTP 1.1 and HTTP 2.0 transparently depending on if the target server supports HTTP 2.0 and is used to call APIs from the following sources:

1. Web Server
2. Standalone Application/Mobile Application
3. API/Service

It provides methods to get authorization tokens and automatically receives the client credentials token for scopes in API to API calls. It also helps to pass correlationId and traceabilityId and other configurable headers to the next service.

HTTP 2.0 is multiplex that one connection can send multiple requests at the same time. There is no need to build a connection pool if only HTTP 2.0 is used. However, some users have services that only support HTTP 1.1 without an immediate path to upgrade. To support those users, we have added a connection pool in the Http2Client. 

Our Http2Client is a very low-level component that is best used in service-to-service communication. 

### Localhost vs 127.0.0.1

When using the client module to call a service on the same host, you can use localhost or 127.0.0.1 before release 1.5.29 as there was a hostname verification bug in the framework. However, from version 1.5.29 onwards, you must use localhost in the URL as the bug was fixed, and localhost matches the hostname in the generated client.truststore and server.keystore files.

If you are using the Http2Client inside a Docker container and the target server is on the host machine or another Docker container, you must use the host IP address instead of localhost. Inside a Docker container, localhost means the Linux OS inside the docker.  

### Generic response callback functions

Like the Undertow core server, it is event-driven with callbacks, so non-blocking all the time to free your CPU for other important computations. Http2Client has two generic callback functions implemented to handle requests with a body (POST, PUT, PATCH) and requests without a body(GET, DELETE). Users can create their own customized callback functions to handle the response from the server if they need special logic inside the callback function.

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

The renewal of the token happens behind the scene, and it supports the circuit breaker if the OAuth 2.0 server is down or busy. The token is renewed pro-actively before the current one expires and lets all requests go with the current token. It only blocks other requests if the current request is trying to renew an expired token. When token renewal in this case fails, all requests will be rejected with a timeout, and subsequent requests will react in the same way until a grace period is passed so that the renewal process starts again.

There is a good reason we renew the token proactively. If we leave the token to expire, the traditional API service can return a 401 error with the expired token message to notify the client to get a new token. In a microservices architecture, this is not possible. What if the token is one second before the expiration, and service A accepts it, writing something into its database which service B accepts, writing something into its database? However, when service C receives it, it is already expired and rejected. There must be extra logic to compensate for the transactions service A and B have performed already. We have a framework [light-kafka][] for this type of microservices orchestration, but it is not necessary for just token expiration handling. It would be better to handle it gracefully in the client module to ensure that the token sent has a longer expiration time than the entire application SLA.

### Sign Request API

The light-oauth2 Token service has a signing endpoint to sign the incoming map object to support the information exchange signature verification between multiple microservices. For more information about the service, please visit [signing][].

For the issuer service which wants to provide the signed request, it needs to use the client module to access the remote service in the light-oauth2. The client module provided an API to simplify access. 

There is a section in the client.yml to define OAuth 2.0 service parameters, and it would be something like the following in the `oauth/token` section. 

For the default config file, please visit [client.yml][]

```
  # sign endpoint configuration
  sign:
    # token server url. The default port number for token service is 6882. If this url exists, it will be used.
    # if it is not set, then a service lookup against serviceId will be taken to discover an instance.
    # server_url: ${client.signServerUrl:https://localhost:6882}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.signProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.signProxyPort:3128}
    # token serviceId. If server_url doesn't exist, the serviceId will be used to lookup the token service.
    serviceId: ${client.signServiceId:com.networknt.oauth2-token-1.0.0}
    # signing endpoint for the sign request
    uri: ${client.signUri:/oauth2/token}
    # timeout in milliseconds
    timeout: ${client.signTimeout:2000}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: ${client.signEnableHttp2:true}
    # client_id for client authentication
    client_id: ${client.signClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
    # client secret for client authentication and it can be encrypted here.
    client_secret: ${client.signClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
    # the key distribution sever config for sign. It can be different then token key distribution server.
    key:
      # key distribution server url. It will be used to establish connection if it exists.
      # if it is not set, then a service lookup against serviceId will be taken to discover an instance.
      # server_url: ${client.signKeyServerUrl:https://localhost:6886}
      # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
      # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
      # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
      # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
      # proxyHost: ${client.signKeyProxyHost:proxy.lightapi.net}
      # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
      # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
      # the default value 443 is going to be used for the HTTP connection.
      # proxyPort: ${client.signKeyProxyPort:3128}
      # the unique service id for key distribution service, it will be used to lookup key service if above url doesn't exist.
      serviceId: ${client.signKeyServiceId:com.networknt.oauth2-key-1.0.0}
      # the path for the key distribution endpoint
      uri: ${client.signKeyUri:/oauth2/key}
      # client_id used to access key distribution service. It can be the same client_id with token service or not.
      client_id: ${client.signKeyClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client secret used to access the key distribution service.
      client_secret: ${client.signKeyClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: ${client.signKeyEnableHttp2:true}
```

Before calling the API, you need to create a POJO object for the SignRequest. The class constructor will load the above configuration parameter into the object. You need to provide these properties:

* expires - which is the number of seconds until the token expires
* payload - which is a map that contains all the attributes you want to put as JWT claims

Once you have the SignRequest object created, you can call the OauthHelper class with the following static method.


```
    public static Result<TokenResponse> getSignResult(SignRequest signRequest) {
```

If there is no error, then the response will be in the TokenResponse. 

### Connection Pool of Http2Client Feature

The high level API is implemented by embedding the connection pool into an existing method:

```java
CompletableFuture<ClientResponse> callService(URI uri, ClientRequest request, Optional<String> requestBody)
```

The user can call this method or `getRequestService(URI uri, ClientRequest request, Optional<String> requestBody)` which combines the `callService()` with a circuit breaker to cache the established connections and reuse them without considering the type of connection.

The low level API is though the borrowConnection and returnConnection methods. The following is the generated test case from the light-codegen.

```java
    @Test
    public void testPetsPetIdDeleteHandlerTest() throws ClientException {

        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            if(enableHttps) {
                connection = client.borrowConnection(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
            } else {
                connection = client.borrowConnection(new URI(url), Http2Client.WORKER, Http2Client.BUFFER_POOL, OptionMap.EMPTY).get();
            }
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        String requestUri = "/v1/pets/VKVDHrCqEjLdRSJEzweRKRPFflj";
        String httpMethod = "delete";
        try {
            ClientRequest request = new ClientRequest().setPath(requestUri).setMethod(Methods.DELETE);
            
            //customized header parameters 
            request.getRequestHeaders().put(new HttpString("key"), "hgNijJaFIMAgwKhUIgnndaLfD");
            request.getRequestHeaders().put(new HttpString("host"), "localhost");
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            client.returnConnection(connection);
        }
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Optional<HeaderValues> contentTypeName = Optional.ofNullable(reference.get().getResponseHeaders().get(Headers.CONTENT_TYPE));
        SchemaValidatorsConfig config = new SchemaValidatorsConfig();
        ResponseValidator responseValidator = new ResponseValidator(config);
        int statusCode = reference.get().getResponseCode();
        Status status;
        if(contentTypeName.isPresent()) {
            status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), contentTypeName.get().getFirst());
        } else {
            status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), JSON_MEDIA_TYPE);
        }
        Assert.assertNull(status);
    }
```

##### HTTP/1.1 Connection
* **Multiple connections** will be added to the connection pool for the same host.
  * This workflow is to monitor whether there are connections that are not yet occupied by other requests and are not waiting for a reponse when a request is sent. If this is the case, reuse the connection until the connection is closed or the maximum number of requests is reached. Otherwise, a new connection is established and cached into the connection pool for this host.

##### HTTP/2 Connection
* **Only one connection** will be added to the connection pool for the same host.
  * When sending a request, reuse it until the connection is closed or the maximum number of requests is reached.


#### Configuration

Users can configure the size of the connection pool, the maximum number of requests per connection, and whether HTTP/2 is enabled through configuring the client.yml.

>Or the user can clean up the connection pool manually by calling `Http2ClientConnectionPool.getInstance().clear()`

*For example:*
```yml
request:
 errorThreshold: 2
 timeout: 3000
 resetTimeout: 7000
 # the flag to indicate whether http/2 is enabled when calling client.callService()
 enableHttp2: true
 # the maximum host capacity of connection pool
 connectionPoolSize: 1000
 # the maximum request limitation for each connection
 maxReqPerConn: 1000000
 # maximum quantity of connection in connection pool for each host
 maxConnectionNumPerHost: 1000
 # minimum quantity of connection in connection pool for each host. The corresponding connection number will shrink to minConnectionNumPerHost
 # by remove least recently used connections when the connection number of a host reach 0.75 * maxConnectionNumPerHost.
 minConnectionNumPerHost: 250
```

#### Example for Fixed URI with callService
```java
try {
    ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(apibPath);
    
    CompletableFuture<ClientResponse> response = client.callService(new URI(apibHost), requestB, Optional.empty());
    
    int statusCodeB = response.join().getResponseCode();
    if (statusCodeB >= 300) {
        throw new Exception("Failed to call API B: " + statusCodeB);
    }
    String apibList = response.join().getAttachment(Http2Client.RESPONSE_BODY);
} catch (Exception e) {
    logger.error("Exception:", e);
    throw new ClientException(e);
}
```

#### Example for Service ID with callService

> Provides an overloading method for callservice() to embedded service discovery and load balancing

```java
try {
    ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(apibPath);
    
    CompletableFuture<ClientResponse> response = client.callService("http", "com.networknt.petstore-3.0.1", "", requestB, Optional.empty());
    
    int statusCodeB = response.join().getResponseCode();
    if (statusCodeB >= 300) {
        throw new Exception("Failed to call API B: " + statusCodeB);
    }
    String apibList = response.join().getAttachment(Http2Client.RESPONSE_BODY);
} catch (Exception e) {
    logger.error("Exception:", e);
    throw new ClientException(e);
}
```

### <span style="color:#0096c8"> Examples </span>

#### Get a client instance

Http2Client is a singleton, and you only need one instance in your application. You can create as many connections from the same instance though. 

```
    Http2Client client = Http2Client.getInstance();
```

#### Call from web server

To set the header with the authorization code JWT token.

```
    public void addAuthToken(HttpRequest request, String token) 
```

To set the header with the authorization code JWT token and traceabilityId.

```
    public void addAuthTokenTrace(HttpRequest request, String token, String traceabilityId) 
```

#### Call from standalone app/mobile app

To set the header with the client credentials JWT token.
```
    public void addCcToken(HttpRequest request) throws ClientException, ApiException 
```

To set the header with the client credentials JWT token and traceabilityId.

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

Within light-4j, there are several places that Http2Client is used to communicate with third party servers:

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

In most cases, we don’t close the connection, and all the requests will go through the same HTTP/2 connection with multiplex support. However, there are times when you need to create a new connection on demand and close it once it is used. For example, in [Server.java](https://github.com/networknt/light-4j/blob/master/server/src/main/java/com/networknt/server/Server.java) we create a new connection to load config from light-config-server during server startup. In this case, although our config server supports HTTP/2, we create and close the connection as this is a one-time request and we don’t want to hold the resource. Here are the lines to close the connection in the final block.

```java
    } finally {
        IoUtils.safeClose(connection);
    }
```

Although it is recommended to keep the connection cached for high volume services, the connection must be closed from time to time so that the load balancer can redirect the same client instance to a different server instance unless you want to pin a particular client instance to a server instance for a long time. This will eventually cause an imbalanced load on the service instances if there are not too many different clients.

The Consul client is one of the examples in which the connection is reset from time to time to ensure that we are not running out of max number of requests per connection in HTTP/2. The current connection will be dropped and recreated after 1 million requests.

https://github.com/networknt/light-4j/blob/master/consul/src/main/java/com/networknt/consul/client/ConsulClientImpl.java

The easier way to close collection periodically is to close it after a number of requests or close it after a number of minutes. The same finally block can be written as:

```java
    } finally {
        if(number of minutes passed or number of requests handled) {
            IoUtils.safeClose(connection);
        }
    }

```

Creating a new connection and closing it after the request is recommended for one-time requests only. However, if you are sending multiple requests from your client during a period of time, it is recommended to use the connection pool with borrowConnection and returnConnection methods.

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

The above code checks the cached connectionC and if it is closed already, creates a new connection and caches it again.

For the full example, please refer to https://github.com/networknt/light-example-4j/blob/master/discovery/api_a/consul/src/main/java/com/networknt/apia/handler/DataGetHandler.java

### Multiple Request Pattern

If you send multiple requests through one connection, here is an example.

```java
    public void testRouterHttps48k() throws Exception {
        ClientConnection connection = null;
        try {
            connection = client.borrowConnection(new URI("https://localhost:8000"), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
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
            client.returnConnection(connection);
        }
    }

```
### Get Request

Setting GET parameters via URL string.

```
ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath("/oauth2/client?page=0&api_key=asdfsomthingelse");
```

You might need to set some headers. 

```
request.getRequestHeaders().put(Headers.HOST, "localhost");
request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
```

### Post Request

I got a lot of questions on how to make a post request to the server, and the above is an example. Please remember the following. 

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

### Request Timeout


`Http2Client` is based on the Undertow client and utilizes callbacks to maximize efficiency. A `CountDownLatch` is passed to the callback function and should be set to zero during the `wait` call once the server response is returned. 

If no timeout is set for the `wait`, the application will wait indefinitely until the response arrives. To avoid this, you can specify a timeout and `TimeUnit` to customize the wait duration. If the timeout is reached but the `CountDownLatch` is not set to zero, it indicates the response is incomplete, and a `ClientException` can be thrown.

Below is an example of a client application that accesses the Petstore API's `/v1/pets` endpoint using the `GET` method:

```java
public class Http2ClientExample {
    static final Logger logger = LoggerFactory.getLogger(Http2ClientExample.class);
    // Get the singleton Http2Client instance
    static Http2Client client = Http2Client.getInstance();

    public static void main(String[] args) {
        Http2ClientExample e = new Http2ClientExample();
        try {
            e.testHttp2Get();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        System.exit(0);
    }

    /**
     * This is a simple example that create a new HTTP 2.0 connection for get request
     * and close the connection after the call is done. As you can see that it is using
     * a hard coded uri which points to a statically deployed service on fixed ip and port
     *
     */
    public void testHttp2Get() throws Exception {
        final CountDownLatch latch = new CountDownLatch(1);
        SimpleConnectionHolder.ConnectionToken connectionToken = null;

        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        boolean completed  = false;
        try {
            connectionToken = client.borrow(new URI("https://localhost:9445"), Http2Client.WORKER, client.getDefaultXnioSsl(), Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true));
            ClientConnection connection = (ClientConnection) connectionToken.getRawConnection();
            ClientRequest request = new ClientRequest().setPath("/v1/pets").setMethod(Methods.GET);
            request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
            request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
            request.getRequestHeaders().put(new HttpString("host"), "localhost");
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            completed = latch.await(ClientConfig.get().getTimeout(), TimeUnit.MILLISECONDS);
        } catch (InterruptedException e) {
            logger.error("InterruptedException:", e);
            throw new ClientException(e);
        } finally {
            client.restore(connectionToken);
        }
        if(!completed) {
            logger.error("Request timeout");
            throw new ClientException("Request timeout");
        } else {
            int statusCode = reference.get().getResponseCode();
            String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
            System.out.println("testHttp2Get: statusCode = " + statusCode + " body = " + body);
        }
    }
}

```

You can find the source code at GitHub [repository](https://github.com/networknt/light-example-4j/blob/master/client/request-timeout/src/main/java/com/networknt/client/Http2ClientExample.java). 


If you are accessing only one target server, you can use the `client.timeout` setting in `values.yml` to configure the wait timeout. For example:

```
latch.await(ClientConfig.get().getTimeout(), TimeUnit.MILLISECONDS);
```

If you have multiple endpoints or services to access, each with different timeout requirements, you can configure their timeouts in your application-specific configuration file.

Additionally, the `timeout` setting in `client.yml` is used to specify the timeout duration for establishing a connection to the server when creating a new connection.

[Here](https://github.com/networknt/light-4j/blob/master/client/src/main/java/com/networknt/client/Http2Client.java#L276) is the usage in Http2Client.


### HOST Header

When using Http2Client, you need to add the following header to the request if HTTP 1.1 is used as it is required for the HTTP 1.1 specification. 

```
request.getRequestHeaders().put(Headers.HOST, "localhost");
```

You can see the entire example in the above testRouterHttps48k method. 


### Configuration

When using the client module of light-4j, you need to have a configuration file client.yml in your src/main/resources/config folder or externalized config folder. 

The following is the default client.yml in light-4j, which you should replace with an externalized client.yml file.

```yaml
# This is the configuration file for Http2Client.
---
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate or load
  # truststore that contains the self-signed certificate.
  verifyHostname: ${client.verifyHostname:true}
  # indicate of system load default cert.
  loadDefaultTrustStore: ${client.loadDefaultTrustStore:true}
  # trust store contains certificates that server needs. Enable if tls is used.
  loadTrustStore: ${client.loadTrustStore:true}
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: ${client.trustStore:client.truststore}
  # trust store password
  trustStorePass: ${client.trustStorePass:password}
  # key store contains client key and it should be loaded if two-way ssl is used.
  loadKeyStore: ${client.loadKeyStore:false}
  # key store location
  keyStore: ${client.keyStore:client.keystore}
  # key store password
  keyStorePass: ${client.keyStorePass:password}
  # private key password
  keyPass: ${client.keyPass:password}
  # public issued CA cert  password
  defaultCertPassword: ${client.defaultCertPassword:changeit}
  # TLS version. Default is TSLv1.2, and you can downgrade to TLSv1 to support some internal old servers that support only TLSv1.1(deprecated
  # and risky). You can also upgrade to TSLv1.3 for maximum security if all your servers support it.
  tlsVersion: ${client.tlsVersion:TLSv1.2}
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  # If there are multiple oauth providers per serviceId, then we need to update this flag to true. In order to derive the serviceId from the
  # path prefix, we need to set up the pathPrefixServices below if there is no duplicated paths between services.
  multipleAuthServers: ${client.multipleAuthServers:false}
  token:
    cache:
      #capacity of caching TOKENs
      capacity: ${client.tokenCacheCapacity:200}
    # The scope token will be renewed automatically 1 minute before expiry
    tokenRenewBeforeExpired: ${client.tokenRenewBeforeExpired:60000}
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: ${client.expiredRefreshRetryDelay:2000}
    # if scope token is not expired but in renew window, we need slow retry delay.
    earlyRefreshRetryDelay: ${client.earlyRefreshRetryDelay:4000}
    # token server url. The default port number for token service is 6882. If this is set,
    # it will take high priority than serviceId for the direct connection
    server_url: ${client.tokenServerUrl:}
    # token service unique id for OAuth 2.0 provider. If server_url is not set above,
    # a service discovery action will be taken to find an instance of token service.
    serviceId: ${client.tokenServiceId:com.networknt.oauth2-token-1.0.0}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    proxyHost: ${client.tokenProxyHost:}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    proxyPort: ${client.tokenProxyPort:}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: ${client.tokenEnableHttp2:true}
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: ${client.tokenAcUri:/oauth2/token}
      # client_id for authorization code grant flow.
      client_id: ${client.tokenAcClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client_secret for authorization code grant flow.
      client_secret: ${client.tokenAcClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # the web server uri that will receive the redirected authorization code
      redirect_uri: ${client.tokenAcRedirectUri:https://localhost:3000/authorization}
      # optional scope, default scope in the client registration will be used if not defined.
      # If there are scopes specified here, they will be verified against the registered scopes.
      # In values.yml, you define a list of strings for the scope(s).
      scope: ${client.tokenAcScope:}
      # - petstore.r
      # - petstore.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: ${client.tokenCcUri:/oauth2/token}
      # client_id for client credentials grant flow.
      client_id: ${client.tokenCcClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client_secret for client credentials grant flow.
      client_secret: ${client.tokenCcClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # optional scope, default scope in the client registration will be used if not defined.
      # If there are scopes specified here, they will be verified against the registered scopes.
      # In values.yml, you define a list of strings for the scope(s).
      scope: ${client.tokenCcScope:}
      # - petstore.r
      # - petstore.w
      # The serviceId to the service specific OAuth 2.0 configuration. Used only when multipleOAuthServer is
      # set as true. For detailed config options, please see the values.yml in the client module test.
      serviceIdAuthServers: ${client.tokenCcServiceIdAuthServers:}
    refresh_token:
      # token endpoint for refresh token grant
      uri: ${client.tokenRtUri:/oauth2/token}
      # client_id for refresh token grant flow.
      client_id: ${client.tokenRtClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client_secret for refresh token grant flow
      client_secret: ${client.tokenRtClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # optional scope, default scope in the client registration will be used if not defined.
      # If there are scopes specified here, they will be verified against the registered scopes.
      # In values.yml, you define a list of strings for the scope(s).
      scope: ${client.tokenRtScope:}
      # - petstore.r
      # - petstore.w
    # light-oauth2 key distribution endpoint configuration for token verification
    key:
      # key distribution server url for token verification. It will be used if it is configured.
      # If it is not set, a service lookup will be taken with serviceId to find an instance.
      server_url: ${client.tokenKeyServerUrl:}
      # key serviceId for key distribution service, it will be used if above server_url is not configured.
      serviceId: ${client.tokenKeyServiceId:com.networknt.oauth2-key-1.0.0}
      # the path for the key distribution endpoint
      uri: ${client.tokenKeyUri:/oauth2/key}
      # client_id used to access key distribution service. It can be the same client_id with token service or not.
      client_id: ${client.tokenKeyClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client secret used to access the key distribution service.
      client_secret: ${client.tokenKeyClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: ${client.tokenKeyEnableHttp2:true}
      # The serviceId to the service specific OAuth 2.0 configuration. Used only when multipleOAuthServer is
      # set as true. For detailed config options, please see the values.yml in the client module test.
      serviceIdAuthServers: ${client.tokenKeyServiceIdAuthServers:}
  # sign endpoint configuration
  sign:
    # token server url. The default port number for token service is 6882. If this url exists, it will be used.
    # if it is not set, then a service lookup against serviceId will be taken to discover an instance.
    # server_url: ${client.signServerUrl:https://localhost:6882}
    # For users who leverage SaaS OAuth 2.0 provider from lightapi.net or others in the public cloud
    # and has an internal proxy server to access code, token and key services of OAuth 2.0, set up the
    # proxyHost here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy server is used.
    # proxyHost: ${client.signProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.signProxyPort:3128}
    # token serviceId. If server_url doesn't exist, the serviceId will be used to lookup the token service.
    serviceId: ${client.signServiceId:com.networknt.oauth2-token-1.0.0}
    # signing endpoint for the sign request
    uri: ${client.signUri:/oauth2/token}
    # timeout in milliseconds
    timeout: ${client.signTimeout:2000}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: ${client.signEnableHttp2:true}
    # client_id for client authentication
    client_id: ${client.signClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
    # client secret for client authentication and it can be encrypted here.
    client_secret: ${client.signClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
    # the key distribution sever config for sign. It can be different then token key distribution server.
    key:
      # key distribution server url. It will be used to establish connection if it exists.
      # if it is not set, then a service lookup against serviceId will be taken to discover an instance.
      # server_url: ${client.signKeyServerUrl:https://localhost:6886}
      # the unique service id for key distribution service, it will be used to lookup key service if above url doesn't exist.
      serviceId: ${client.signKeyServiceId:com.networknt.oauth2-key-1.0.0}
      # the path for the key distribution endpoint
      uri: ${client.signKeyUri:/oauth2/key}
      # client_id used to access key distribution service. It can be the same client_id with token service or not.
      client_id: ${client.signKeyClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
      # client secret used to access the key distribution service.
      client_secret: ${client.signKeyClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: ${client.signKeyEnableHttp2:true}
  # de-ref by reference token to JWT token. It is separate service as it might be the external OAuth 2.0 provider.
  deref:
    # Token service server url, this might be different than the above token server url. The static url will be used if it is configured.
    # server_url: ${client.derefServerUrl:https://localhost:6882}
    # For users who leverage SaaS OAuth 2.0 provider in the public cloud and has an internal
    # proxy server to access code, token and key services of OAuth 2.0, set up the proxyHost
    # here for the HTTPS traffic. This option is only working with server_url and serviceId
    # below should be commented out. OAuth 2.0 services cannot be discovered if a proxy is used.
    # proxyHost: ${client.derefProxyHost:proxy.lightapi.net}
    # We only support HTTPS traffic for the proxy and the default port is 443. If your proxy server has
    # a different port, please specify it here. If proxyHost is available and proxyPort is missing, then
    # the default value 443 is going to be used for the HTTP connection.
    # proxyPort: ${client.derefProxyPort:3128}
    # token service unique id for OAuth 2.0 provider. Need for service lookup/discovery. It will be used if above server_url is not configured.
    serviceId: ${client.derefServiceId:com.networknt.oauth2-token-1.0.0}
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: ${client.derefEnableHttp2:true}
    # the path for the key distribution endpoint
    uri: ${client.derefUri:/oauth2/deref}
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: ${client.derefClientId:f7d42348-c647-4efb-a52d-4c5787421e72}
    # client_secret for deref
    client_secret: ${client.derefClientSecret:f6h1FTI8Q3-7UScPZDzfXA}
# If you have multiple OAuth 2.0 providers and use path prefix to decide which OAuth 2.0 server
# to get the token or JWK. If two or more services have the same path, you must use serviceId in
# the request header to use the serviceId to find the OAuth 2.0 provider configuration.
pathPrefixServices: ${client.pathPrefixServices:}
# circuit breaker configuration for the client
request:
  # number of timeouts/errors to break the circuit
  errorThreshold: ${client.errorThreshold:2}
  # timeout in millisecond to indicate a client error.
  timeout: ${client.timeout:3000}
  # reset the circuit after this timeout in millisecond
  resetTimeout: ${client.resetTimeout:7000}
  # if open tracing is enabled. traceability, correlation and metrics should not be in the chain if opentracing is used.
  injectOpenTracing: ${client.injectOpenTracing:false}
  # inject serviceId as callerId into the http header for metrics to collect the caller. The serviceId is from server.yml
  injectCallerId: ${client.injectCallerId:false}
  # the flag to indicate whether http/2 is enabled when calling client.callService()
  enableHttp2: ${client.enableHttp2:true}
  # the maximum host capacity of connection pool
  connectionPoolSize: ${client.connectionPoolSize:1000}
  # Connection expire time when connection pool is used. By default, the cached connection will be closed after 30 minutes.
  # This is one way to force the connection to be closed so that the client-side discovery can be balanced.
  connectionExpireTime: ${client.connectionExpireTime:1800000}
  # The maximum request limitation for each connection in the connection pool. By default, a connection will be closed after
  # sending 1 million requests. This is one way to force the client-side discovery to re-balance the connections.
  maxReqPerConn: ${client.maxReqPerConn:1000000}
  # maximum quantity of connection in connection pool for each host
  maxConnectionNumPerHost: ${client.maxConnectionNumPerHost:1000}
  # minimum quantity of connection in connection pool for each host. The corresponding connection number will shrink to minConnectionNumPerHost
  # by remove least recently used connections when the connection number of a host reach 0.75 * maxConnectionNumPerHost.
  minConnectionNumPerHost: ${client.minConnectionNumPerHost:250}
``` 

The values of `keystore` and `truststore` also represent the paths. They should be relative to the external config directory. For example, If the server start with `-Dlight-4j-config-dir=/external_dir/config`, and the absolute path of `keystore` is `/external_dir/config/ssl/client.keystore`, their configuration in client.yml should be:

```yaml
tls:
  ...
  trustStore: ssl/client.truststore
  ...
  keyStore: ssl/client.keystore
  ...
```

Otherwise, if all config files are flattened into one folder. The configuration could be simplified as:

```yaml
tls:
  ...
  trustStore: client.truststore
  ...
  keyStore: client.keystore
  ...
```

And here is an example of secret.yaml with client secrets in 1.6.x release. For 2.0.x release, we don't use the secret.yml anymore and all sensitive info are in the client.yml encryped. 

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
[light-kafka]: /style/light-kafka/
[signing]: /service/oauth/service/signing/
[client.yml]: https://github.com/networknt/light-4j/blob/master/client/src/main/resources/config/client.yml
