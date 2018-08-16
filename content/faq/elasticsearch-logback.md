---
title: "Elasticsearch Logback Template"
date: 2018-08-16T09:16:06-04:00
description: "How to handle Java stacktrace in logback template to work with ElasticSearch?"
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When using ElasticSearch for centralized logging, all logback output will goes to the stdout so that Logstash or Fluentd will pick it up and inject into ElasticSearch for indexing. In the production logback.xml in externalized config folder, there is a template with fixed columns to assist log injection. Everything works except Java stacktrace as it contains multiple lines and all injectors are handle single line only. 

To make multiple line stacktrace into a single line, we have manipulated the logback template a bit to concat the stacktrace. 

Here is an exmaple of the logback.xml and this should only be used on environment with centralized logging. If you want to look at the logs in file or from the console, use the logback.xml in the resources folder. 

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <turboFilter class="ch.qos.logback.classic.turbo.MarkerFilter">
        <Marker>PROFILER</Marker>
        <!--<OnMatch>DENY</OnMatch>-->
        <OnMatch>NEUTRAL</OnMatch>
    </turboFilter>

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %X{sId} %X{cId} %X{appNumber} %-5level %logger{36} %M - %msg %replace(%ex){'[\r\n]+', '\\n'}%nopex%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="stdout"/>
    </root>

</configuration>
```
