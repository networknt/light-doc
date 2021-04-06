---
title: "Correlation Id"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

CorrelationId handler is part of the [traceability][] of microservices architecture along with the traceabilityId handler that is used to index centralized logs collected from all containerized services or the original client.

This is a middleware handler that checks if X-Correlation-Id exists in the request header. If it doesn't exist it will generate a UUID and put it into the request header. During API to API calls, this header will be passed to the next API by the [Client][] module.

### Generating

The correlationId is very useful in microservices architecture as there are multiple services involved in the same client request. When logs are aggregated into a centralized tool like Elasticsearch, it is very important there is a unique identifier to associate logs from multiple services for the same request. The Id is a UUID and must be generated in the first service called from the original client.

The generation logic is very simple. Whenever the correlationId handler is wired in the request/response chain, and it is enabled and autogenCorrelationID is true in the correlation.yml config file, it checks if X-Correlation-Id is in the request header. If the header doesn’t exist, a UUID will be created. When all microservices have this handler enabled, the first microservice in the invocation chain generates the correlationId. This correlationId will be passed to all the subsequent microservices after it is generated. This will ensure that all services use the same correlationId for logging.

### Passing

Since the first service generates the Id, it must be passed to other services somehow so that subsequent services can use it to log their messages. In our [client][] module, it passes the correlationId from the current request header to the request to the next service.

In Http2Client, the following method and its dependency are the most used ones for service to service invocation. It passes traceabilityId, correlationId and the original JWT token to the next client request. It also populates the scope-token cached in the client module to the next request to indicate the immediate caller. 

```
    /**
     * Support API to API calls with scope token. The token is the original token from consumer and
     * the client credentials token of caller API is added from cache.
     *
     * This method is used in API to API call
     *
     * @param request the http request
     * @param exchange the http server exchange
     */
    public Result propagateHeaders(ClientRequest request, final HttpServerExchange exchange) {
        String tid = exchange.getRequestHeaders().getFirst(HttpStringConstants.TRACEABILITY_ID);
        String token = exchange.getRequestHeaders().getFirst(Headers.AUTHORIZATION);
        String cid = exchange.getRequestHeaders().getFirst(HttpStringConstants.CORRELATION_ID);
        return populateHeader(request, token, cid, tid);
    }

    /**
     * Support API to API calls with scope token. The token is the original token from consumer and
     * the client credentials token of caller API is added from cache. authToken, correlationId and
     * traceabilityId are passed in as strings.
     *
     * This method is used in API to API call
     *
     * @param request the http request
     * @param authToken the authorization token
     * @param correlationId the correlation id
     * @param traceabilityId the traceability id
     * @return Result when fail to get jwt, it will return a Status.
     */
    public Result populateHeader(ClientRequest request, String authToken, String correlationId, String traceabilityId) {
        if(traceabilityId != null) {
            addAuthTokenTrace(request, authToken, traceabilityId);
        } else {
            addAuthToken(request, authToken);
        }
        Result<Jwt> result = tokenManager.getJwt(request);
        if(result.isFailure()) { return Failure.of(result.getError()); }
        request.getRequestHeaders().put(HttpStringConstants.CORRELATION_ID, correlationId);
        request.getRequestHeaders().put(HttpStringConstants.SCOPE_TOKEN, "Bearer " + result.getResult().getJwt());
        return result;
    }
```

### Logging

This handler gets the X-Correlation-Id from the request header or generates one if it doesn't exist in the request header. After that, it puts it into the org.slf4j.MDC so that `logback` can put it into the log for every logging statement. 

If the logging statement is from the same thread of the correlationId handler, it works perfectly; however, we still have an issue if the business handler starts other threads. Here is the GitHub [issue](https://github.com/networknt/light-4j/issues/193) with a detailed description. 

### logback.xml

In the generated logback.xml, the cId is part of the appender config as pattern "%X{cId}"

```
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %X{cId} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="log" class="ch.qos.logback.core.FileAppender">
        <File>target/test.log</File>
        <Append>false</Append>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%d{HH:mm:ss.SSS} [%thread] %X{cId} %-5level %class{36}:%L %M - %msg%n</Pattern>
        </layout>
    </appender>

```
### Configuration

The configuration for this module is very simple. Just enable it or not. Here is the default config named correlation.yml in the module. You can externalize this config file and turn the handler autogenCorrelationID off if necessary.

```
# Correlation Id Handler Configuration
---
# If enabled is true, the handler will be injected into the request and response chain.
enabled: true

# If set to true, it will auto-generate the correlationID if it is not provided in the request
autogenCorrelationID: true
```

### TraceabilityId

For most applications, correlation and traceability handlers are used together. This allows users to associate the traceabilityId with the correlationId as correlationId will be in every logging statement. If the correlation handler is used correctly, there is only one service in the chain to be responsible for generating the correlationId, and a logging statement will log both traceabilityId and correlationId together for cross-reference. 

Please visit [traceability handler][] for more information. 

[Client]: /concern/client/
[traceability]: /architecture/traceability/
[traceability handler]: /concern/traceability/
