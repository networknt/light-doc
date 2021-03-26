---
title: "Audit Log"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Two built-in audit handlers write logs into audit.log which is set up in the logback appender. The end user can add more customized audit handlers if needed. For example, some customers are writing audit info in a relational database, while others write it into a Kafka topic.

In the audit module, there is AuditHandler which is generic and configurable with the audit.yml config file. 

There is another audit provided by the light-4j framework called DumpHandler in the [dump](concern/dump/) module.

### Introduction 

This handler only logs several fields from the request header, while the fields are configurable. Optionally, it can log response status and response time.

This is a generic audit handler that dumps the most important info per request basis. The following elements will be logged if they are available in the AuditInfo object attached to exchange. This object will be populated by other upstream handlers like swagger-meta and swagger-security for the light-rest-4j framework.

This handler can be used on production, but be aware that it will impact overall performance. Turning off statusCode and responseTime can make it faster, as these have to be captured on the response chain instead of request chain.

For most business and majority of microservices, you don't need to enable this handler due to performance reasons. There are situations during troubleshooting or integration testing, where teams need to collect more information based on the success or failure of the execution or collect complete request and response payloads. Teams need to recognize that collecting this information will negatively impact performance and it is not recommended to use it on production environments.

The default audit log will be audit.log config in the default logback.xml; however, it can be changed to syslog or Kafka with a customized appender.

The majority of the fields in the audit log are collected in request and response; however, to allow users to customize it, we have put an attachment into the exchange to allow other handlers to write important info into it. The audit.yml can control which fields should be included in the final log.

By default, the following fields are included:

 * timestamp
 * serviceId (from server.yml)
 * correlationId
 * traceabilityId (if available)
 * clientId
 * userId (if available)
 * scopeClientId (available if called by another API)
 * endpoint (uriPattern@method)
 * statusCode
 * responseTime
 * request (the request body, if available)
 * response (the response payload, if set)

The audit.log is in JSON format, and it is easy to be parsed and monitored. 

### Configuration
You will need to define com.networknt.audit.AuditHandler@audit in handler.yml then add this “audit” handler to the handler chain.
 
The output fields are populated based on the config file audit.yml. Here is an example:

```
# AuditHandler will pick some important fields from headers and tokens and logs into a audit appender.
---
# Enable Audit
enabled: true

# Enable mask
mask: true

# Output response status code
statusCode: true

# Output response time
responseTime: true

# when auditOnError is true:
#  - it will only log when status code >= 400
#  - response body will be only logged when auditOnError is true
#  - status detail will be only logged when auditOnError is true
# when auditOnError is false:
#  - it will log on every request
#  - no response body will be logged
#  - no status detail will be logged
# log level is controlled by logLevel
auditOnError: false

# will audit stacktrace when using default setExchangeStatus
# auditStackTrace: false

# log level; by default set to info
logLevelIsError: false

# the format for outputting the timestamp, if the format is not specified or invalid, will use a long value.
timestampFormat: yyyy-MM-dd'T'HH:mm:ss.SSSZ

# Output header elements. You can add more if you want.
headers:

# Correlation Id
- X-Correlation-Id

# Traceability Id
- X-Traceability-Id

# Output from id token and access token
audit:

# Client Id
- client_id

# User Id in id token, this is optional
- user_id

# Client Id in scope/access token, this is optional
- scope_client_id

# Request endpoint uri@method.
- endpoint

# Service ID assigned to the service, this is optional and must be set by the service in its implementation
- serviceId

# Request Body, this is optional and must be set by the service in its implementation
- requestBody

# Request Cookies, this is optional and must be set by the service in its implementation
- requestCookies

# Request Query Parameters, this is optional and must be set by the service in its implementation
- queryParameters

# Request Path Parameters, this is optional and must be set by the service in its implementation
- pathParameters

# Response payload, this is optional and must be set by the service in its implementation
- responseBody

# Response error status payload
- Status

```

### Logback config

The following is the appender defined in the logback.xml or logback-test.xml

```
    <!--audit log-->
    <appender name="audit" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>target/audit.log</file> <!-- logfile location -->
        <encoder>
            <pattern>%-5level [%thread] %date{ISO8601} %F:%L - %msg%n
            </pattern> <!-- the layout pattern used to format log entries -->
            <immediateFlush>true</immediateFlush>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>target/audit.log.%i.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>5</maxIndex> <!-- max number of archived logs that are kept -->
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>200MB
            </maxFileSize> <!-- The size of the logfile that triggers a switch to a new logfile, and the current one archived -->
        </triggeringPolicy>
    </appender>

```

### Logging example
Below is the example with default configs:

```
INFO  [XNIO-1 I/O-1] 2017-05-08 19:32:33,975 AuditHandler.java:141 - {"timestamp":1494286353929,"X-Correlation-Id":"abS_cAyTT5SIrHayM-11pQ","X-Traceability-Id":null,"statusCode":200,"responseTime":16}
INFO  [XNIO-1 I/O-3] 2017-05-08 19:32:33,975 AuditHandler.java:141 - {"timestamp":1494286353960,"X-Correlation-Id":"FUD_bbFpRpSs2CmVjJYt-A","X-Traceability-Id":"tid","statusCode":200,"responseTime":1}
```
With different timestamp format, request body, query parameters:
```
16:22:07.112 [XNIO-1 task-1]  KUgvy0fqQNGO_8mCnnj1Vw INFO  Audit lambda$handleRequest$0 - {"timestamp":"2020-12-14T16:22:02.719-0500","X-Correlation-Id":"KUgvy0fqQNGO_8mCnnj1Vw","X-Traceability-Id":"LZIaEFAipVQvSTTEz","requestBody":"{\"name\":\"pepper\",\"type\":\"doodle\"}","queryParameters":"{parm1=[v1]}","statusCode":200,"responseTime":4389}

```

### Customized Handler

For some users that need special audit logic or other channels to redirect the audit to, they can create their audit handler and replace the default audit handler in server.yml or handler.yml depending on how you wire your middleware handlers in your application. 
