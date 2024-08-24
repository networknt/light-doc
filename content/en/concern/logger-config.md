---
title: "Logger Config"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Logger-config is a module in the Light-4j framework that will be used to get the loggers and their current logging levels. It can also change the logging level for given loggers at runtime (Example: Change logging level to DEBUG for “com.networknt” logger for troubleshooting purposes). The user can also create a brand new logger with a level for debugging issues for a specific package or class on the target server.

**Note:** This feature is only supposed to be used in a development environment without security enabled. On production, the security must be enabled so only authorized users can update the logging level through API calls. It is highly recommended to leverage the light controller (standard) or light portal (enterprise) to manage all the runtime instances and update them through the UI.

### Specification

The specification is injected to any light-rest-4j application via [openapi-inject.yml](https://github.com/networknt/light-rest-4j/blob/master/openapi-meta/src/main/resources/config/openapi-inject.yml) when the application specific openapi.yaml file is loaded. 

Here are the endpoint definitions.

```
  /adm/logger:
    get:
      description: to get the current logging settings
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
    post:
      description: to modify the logging settings
      requestBody:
        description: to update logging settings
        content:
          application/json:
            schema:
              type: "array"
              items:
                type: "object"
                properties:
                  name:
                    type: "string"
                  level:
                    type: "string"
        required: true
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin

  /adm/logger/content:
    get:
      description: to get the content of the logs from this service.
      security:
        - admin-scope:
            - admin
            - ${server.serviceId}/admin

```

The first endpoint /adm/logger@get is used to get all the loggers configured in the logback.xml on the target server. 

The second endpoint /adm/logger@post is used to update existing loggers or add new loggers. The body of the request is an array of maps in JSON format. Each map will have the key as the logger name and the value as the logging level.

The third endpoint /adm/logger/content@get is used to get the logs from the server. It is used by the control pane and the light portal UI to display the logs from the target server.

### Configuration

If you use light-codegen to generate a light-rest-4j application, the handler.yml will have this endpoint configured. Also, you can find the examples in http-sidecar and light-gateway repositories. 

handler.yml

```
paths:
  .
  .
  .
  - path: '/adm/logger'
    method: 'get'
    exec:
      - admin
      - getLogger

  - path: '/adm/logger'
    method: 'post'
    exec:
      - admin
      - postLogger

  - path: '/adm/logger/content'
    method: 'GET'
    exec:
      - admin
      - getLogContents
   .
   .
   .

```

If the handler.chains.default is externalized, you need to register the handler classes and give aliases to them in the values.yml file.

values.yml

```
handler.handlers:
  .
  .
  .
  - com.networknt.logging.handler.LoggerGetHandler@getLogger
  - com.networknt.logging.handler.LoggerPostHandler@postLogger
  - com.networknt.logging.handler.LoggerGetLogContentsHandler@getLogContents
```

The logger-config module has its configuration to control if the module is enabled or not. 

Here is the default logging.yml

```
---
# Logging endpoint that can output the logger with logging levels.

# Indicate if the logging info is enabled or not.
enabled: ${logging.enabled:true}
# Indicate the default time period backward in millisecond for log content retrieve. Default is an hour which indicate system will retrieve one hour log by default
logStart: ${logging.logStart:600000}

# if the logger access needs to invoke down streams API. It is false by default.
downstreamEnabled: ${logging.downstreamEnabled:false}
# down stream API host. http://localhost is the default when used with http-sidecar and kafka-sidecar.
downstreamHost: ${logging.downstreamHost:http://localhost:8081}
# down stream API framework that has the admin endpoints. Light4j, SpringBoot, Quarkus, Micronaut, Helidon, etc. If the adm endpoints are different between
# different versions, you can use the framework plus version as the identifier. For example, Light4j-1.6.0, SpringBoot-2.4.0, etc.
downstreamFramework: ${logging.downstreamFramework:Light4j}
```

You can disable the logger-config module and change the log retrieval timestamp in the values.yml file to overwrite the default values. 


### Security

Depending on how the security is configured on the server, you need to get a token and put it into the Authorization header to access the logger endpoints. If the light portal is used, the request can be sent from the control pane to each individual service instance. When a user logs in to the light portal, a token will be created with the proper scope to access the admin endpoints. 

You can disable the security in the values.yml file for a simple local test. 

values.yml

```
# security.yml
security.enableVerifyJwt: false
```

### Dependency

Add dependency in the application pom.xml file.

```xml
<dependency>
  <groupId>com.networknt</groupId>
  <artifactId>logger-handler</artifactId>
  <version>${version.light-4j}</version>
</dependency>
```

### Usage

By default, the logger-config module is enabled in logging.yml. If you want to disable it, just add an entry "logging.enabled: false" in values.yml to overwrite the default value to false. 

To get the loggers and their levels. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger'
```

And the result should be something like. 

```
[
    {
        "name": "ROOT",
        "level": "ERROR"
    },
    {
        "name": "Audit",
        "level": "INFO"
    },
    {
        "name": "com.networknt",
        "level": "TRACE"
    }
]
```

To change the loging level for com.networknt to ERROR and add another logger "com.networknt.server.Server" to TRACE.

```
curl -k --location --request POST 'https://localhost:8443/adm/logger' \
--header 'Content-Type: application/json' \
--data-raw '[
    {
        "name": "com.networknt",
        "level": "ERROR"
    },
    {
        "name": "com.networknt.handler.ProxyHandler",
        "level": "TRACE"
    }
]'
```
And the result is.

```
[
    {
        "name": "com.networknt",
        "level": "ERROR"
    },
    {
        "name": "com.networknt.handler.ProxyHandler",
        "level": "TRACE"
    }
]
```

On the server, the trace should be enabled for the com.networknt.server.Server class and we can get the logs with the following default command. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content'
```

The result should be something like below. 

```
{
    "ROOT": {
        "total": 1,
        "logs": [
            {
                "timestamp": "2022-11-11T14:58:21.880-0500",
                "thread": "XNIO-1 task-1",
                "level": "ERROR",
                "logger": "com.networknt.handler.LightHttpHandler",
                "correlationId": "toseSd8CTg2a1aECCYKkqA",
                "serviceId": "",
                "class": "LightHttpHandler.java",
                "lineNumber": "com.networknt.handler.LightHttpHandler:116",
                "method": "setExchangeStatus",
                "logMessage": {
                    "statusCode": 401,
                    "code": "ERR10002",
                    "message": "MISSING_AUTH_TOKEN",
                    "description": "No Authorization header or the token is not bearer type",
                    "severity": "ERROR"
                }
            }
        ]
    },
    "com.networknt": {
        "total": 1,
        "logs": [
            {
                "timestamp": "2022-11-11T14:58:21.880-0500",
                "thread": "XNIO-1 task-1",
                "level": "ERROR",
                "logger": "com.networknt.handler.LightHttpHandler",
                "correlationId": "toseSd8CTg2a1aECCYKkqA",
                "serviceId": "",
                "class": "LightHttpHandler.java",
                "lineNumber": "com.networknt.handler.LightHttpHandler:116",
                "method": "setExchangeStatus",
                "logMessage": {
                    "statusCode": 401,
                    "code": "ERR10002",
                    "message": "MISSING_AUTH_TOKEN",
                    "description": "No Authorization header or the token is not bearer type",
                    "severity": "ERROR"
                }
            }
        ]
    }
}
```

The default logging level is ERROR and we can change it to other levels in the query parameter. Also, we can choose the logger we need to get the output. The following commmand will only get logs for logger com.networknt and the level is trace. 


```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE'
```

The result should be something like below. 

```
{
    "com.networknt": {
        "total": 92,
        "logs": [
            {
                "timestamp": "2022-11-11T17:28:28.829-0500",
                "thread": "XNIO-1 task-1",
                "level": "DEBUG",
                "logger": "com.networknt.limit.RateLimiter",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RateLimiter.java",
                "lineNumber": "com.networknt.limit.RateLimiter:250",
                "method": "isAllowByServer",
                "logMessage": "CurrentTimeWindow:1668205708 Result:true  Count:0"
            },
            {
                "timestamp": "2022-11-11T17:28:28.830-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.openapi.JwtVerifyHandler",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "JwtVerifyHandler.java",
                "lineNumber": "com.networknt.openapi.JwtVerifyHandler:96",
                "method": "handleRequest",
                "logMessage": "Skip request path base on skipPathPrefixes for /v1/pets"
            },
            {
                "timestamp": "2022-11-11T17:28:28.831-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.reqtrans.RequestTransformerInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestTransformerInterceptor.java",
                "lineNumber": "com.networknt.reqtrans.RequestTransformerInterceptor:77",
                "method": "handleRequest",
                "logMessage": "RequestTransformerHandler.handleRequest is called."
            },
            {
                "timestamp": "2022-11-11T17:28:28.833-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.body.RequestBodyInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestBodyInterceptor.java",
                "lineNumber": "com.networknt.body.RequestBodyInterceptor:73",
                "method": "handleRequest",
                "logMessage": "RequestBodyInterceptor is called."
            },
            {
                "timestamp": "2022-11-11T17:28:28.834-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.body.RequestBodyInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestBodyInterceptor.java",
                "lineNumber": "com.networknt.body.RequestBodyInterceptor:119",
                "method": "shouldAttachBody",
                "logMessage": "hasBody = false isPathConfigured = true"
            },
            .
            .
            .

```

You can change the pagination limit and offset from default 100 and 0. 

For example. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE&limit=10&offset%20=10'
```

Above command will get 10 log statements starting from line 10. You can specify startTime and endTime in epoch milliseconds to get logs for a specific period. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE&startTime=1668000000000&endTime=1668206563000'
```


### Pass Through

When the http-sidecar is used for backend API implemented with other frameworks and languages, get loggers and set loggers should be passed through to the backend API to make the logger configuration changes on the backend API instance. 

If the backend API is a light-4j server, the request can be passed through directly without any transformation. If the backend is another framework, we might need to transform the request path, headers, query parameters and body if necessary. The following uses Spring Boot actuator endpoint as examples. 


The backend spring boot application can be found at https://github.com/networknt/light-example-4j/tree/release/springboot/log

Here is the command line to start the spring boot application. 

```
steve@lucky:~/networknt/light-example-4j/springboot/log$ mvn spring-boot:run --debug
```

The following endpoint outputs some logging statements in stdout to indicate the logging level. 

```
curl http://localhost:8080
```
The default result in the stdout should look like the following to indicate the logging level is INFO.

```
2023-06-21 15:53:43.404  INFO 36112 --- [nio-8080-exec-1] com.networknt.log.LoggingController      : An INFO Message
2023-06-21 15:53:43.404  WARN 36112 --- [nio-8080-exec-1] com.networknt.log.LoggingController      : A WARN Message
2023-06-21 15:53:43.404 ERROR 36112 --- [nio-8080-exec-1] com.networknt.log.LoggingController      : An ERROR Message
```

You can use the following API to get the logging levels for all loggers on the Spring Boot App. 

```
curl http://localhost:8080/actuator/loggers
```
Here is the partial of the result.

```
{"levels":["OFF","ERROR","WARN","INFO","DEBUG","TRACE"],"loggers":{"ROOT":{"configuredLevel":"INFO","effectiveLevel":"INFO"},"_org":{"configuredLevel":null,"effectiveLevel":"INFO"},"_org.springframework":{"configuredLevel":null,"effectiveLevel":"INFO"},
.
.
.
```

To change the logging level for the ROOT logger, you can use the following curl command. 

```
curl --location 'http://localhost:8080/actuator/loggers/ROOT' \
--header 'Content-Type: application/json' \
--data '{
    "configuredLevel": "TRACE"
}'
```

After the logging level is changed, the same API endpoint http://localhost:8080 will have the following output in stdout to indicate the logging level is TRACE. 

```
2023-06-21 16:17:08.363 TRACE 36112 --- [nio-8080-exec-9] com.networknt.log.LoggingController      : A TRACE Message
2023-06-21 16:17:08.363 DEBUG 36112 --- [nio-8080-exec-9] com.networknt.log.LoggingController      : A DEBUG Message
2023-06-21 16:17:08.363  INFO 36112 --- [nio-8080-exec-9] com.networknt.log.LoggingController      : An INFO Message
2023-06-21 16:17:08.363  WARN 36112 --- [nio-8080-exec-9] com.networknt.log.LoggingController      : A WARN Message
2023-06-21 16:17:08.363 ERROR 36112 --- [nio-8080-exec-9] com.networknt.log.LoggingController      : An ERROR Message

```

We are familiar with the Spring Boot actuator endpoints for loggers. With property configuration on the http-sidecar, we can access the above endpoints from the /adm/logger API light-4j on the sidecar. 

The following are the configuration properties in values.yml for the Spring Boot backend with the loggers actuator enabled.

```
# logging.yml
logging.downstreamEnabled: true
logging.downstreamHost: http://localhost:8080
logging.downstreamFramework: SpringBoot
```

The entire configuration for the http-sidecar can be found at https://github.com/networknt/http-sidecar/tree/master/config/actuator


Once the http-sidecar is started with the actuator configuration with -Dlight-4j-config-dir=config/actuator option, we can access the backend actuator endpoint with the following commands with a special header X-Adm-Passthrough. 

Get logging level for loggers.

```
curl --location 'https://localhost:9445/adm/logger' \

--header 'X-Adm-Passthrough: true'
```
 Update logging level for loggers.

```
curl --location 'https://localhost:9445/adm/logger' \
--header 'X-Adm-Passthrough: true' \
--header 'Content-Type: application/json' \
--data '[
    {
        "name": "ROOT",
        "level": "DEBUG"
    },
    {
        "name": "com.networknt",
        "level": "TRACE"
    }
]'
```

As you can see, we are using the same commands to get and post loggers for the light-4j instance, like the http-sidecar and the backend API, like the spring boot application. The only difference is the header X-Adm-Passthrough exists with true value or not. 


### Light-Controller

The light-controller is using only the getLogger and postLogger from the UI for each registered instance. For more info, please visit [light-controller](https://doc.networknt.com/service/controller/).


{{< youtube 5iHRD8sRKY0 >}}
