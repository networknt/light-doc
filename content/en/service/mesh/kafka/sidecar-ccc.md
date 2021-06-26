---
title: "Sidecar Cross-cutting Concerns"
date: 2021-04-14T10:08:53-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The Kafka Sidecar is built with light-4j and light-rest-4j, so all the common cross-cutting concerns of [light-4j](/concern/) and [light-rest-4j](/style/light-rest-4j/) can be enabled with configuration. The cross-cutting concerns below are specific for the Kafka Sidecar only.


### Dead Letter

There are two situations we will use the dead letter topic. One is on the sidecar with the reactive consumer, and the other is on the backend API with the active consumer. 

There are two configurable properties for the dead letter process: the deadLetterEnabled to control if the sidecar has the dead letter process enabled or not. The deadLetterTopicExt is the ext that is added to the original topic for the dead letter topic name. 

```
# Indicator if the dead letter topic is enabled.
deadLetterEnabled: ${kafka-consumer.deadLetterEnabled:true}
# The extension of the dead letter queue(topic) that is added to the original topic to form the dead letter topic
deadLetterTopicExt: ${kafka-consumer.deadLetterTopicExt:.dlq}
```

##### Reactive Consumer

The sidecar will retrieve a batch of records for the reactive consumer and send them to the backend API for processing. There are three situations that we need to handle differently. 

* Cannot connect to the backend API as it is down. 
* Connected but got timeout or 400 or 500 errors that indicate it is overloaded or got system-wide exceptions.
* Successfully sent/processed the records, and the backend API returns 200 errors with a list of RecordProcessedResult. 

In the scenario 1 and 2, we need to keep trying after waiting for a period of time that based on the configuration in the kafka-consumer.yml; however, in the scenario 3, we need to parse the response list and push the un-processed result to the dead letter topic. 

For each message that is pushed to the dead letter topic, the original message sent to the backend API is recorded. Also, a stack trace was captured from the backend API to help developers fix the un-caught exception on the backend API. 

To replay the dead letter topic, we need to allow the backend API to be fixed and redeployed. Once this is done, we can invoke a sidecar API to replay the dead letters through the Control Pane. 


##### Active Consumer

With an active consumer, the backend API has the control on when to load the records from the sidecar. If there are any records that cannot be processed due to an exception, it will invoke a sidecar API to push the problematic record to the dead letter topic through the sidecar. Given the backend API has the knowledge on which record(s) cannot be processed successfully, it should only push the problematic record(s) to the dead letter topic. 


##### Replay

When the dead letter message is rewritten to the dead letter topic, the original message that the sidecar sent to the backend API is recorded; when we replay the dead letter messages, we don't need to retrieve it from the original topic. Sometimes, the original topic retention.ms is not long enough. The original message might be gone already when we replay the dead letter message. It is why we keep the original message in the dead letter topic for replay instead of referring to the original topic. 

Each backend API will have a separate dead letter topic that is named with the original topic + an ext that is defined in the kafka-consumer.yml file. For each onboarding consumer, we need to create a dead letter topic for it. And it will put the ext into the config file to construct the dead letter topic name in the sidecar. By setting up each backend API a separated dead letter topic, we can avoid putting the consumer group to the dead letter message. Also, we don't need to maintain the replay state in a separate state topic to manage the offset for each backend API. 

```
# The extension of the dead letter queue(topic) that is added to the original topic to form the dead letter topic
deadLetterTopicExt: ${kafka-consumer.deadLetterTopicExt:.dl1}
```

The replay will be an endpoint exposed on the sidecar, and it can be accessed from the Control Pane like other admin endpoints. The replay will always start from the offset 0; however, the user can choose the last offset to replay. If there is no last offset specified in the request, it will replay all messages in the dead letter topic. 

Some dead letter messages still cannot be handled during the replay process, and they will be pushed to the dead letter topic again. The replay process won't process the newly produced dead-letter messages as their offset is greater than the offset specified for the request. 

When sidecar replays the dead letter messages for its specific dead letter topic, it starts another separate consumer group. The Kafka consumer group will manage the offset to avoid repeatedly replaying the same message. The offset will be advanced for each replay process, and the next replay will always start from the last completed offset + 1. 



Questions:

1. Do we need to have the same number of partitions for the dead letter topic? If yes, do we need to push the replay messages to the right instance for the partition matching the original topic? 

2. If the above question is answered yes, we need to use the original message key as the dead letter topic key and design the same schema as the original topic key. 

3. The only scenario that we need to make sure that the backend API pin to a particular partition is due to the physical separation. For example, one instance is processing North American claims, and the other is processing Asian claims. 

### Audit

Audit on the sidecar is an important cross-cutting concerns that we need to address. Every records we send to the backend API and the response from the backend API should be persisted in a sidecar audit log file or Kafka topic and potentially push to a SQL or No-SQL database for query and reporting. We can also use streams processing on the Control Pane to project the audit info to different dementions for query and report from the Control Pane. 

Here is the section about audit in the kafka-consumer.yml 

```
# Indicator if the audit is enabled.
auditEnabled: ${kafka-consumer.auditEnabled:true}
# Audit log destination topic or logfile. Default to topic
auditTarget: ${kafka-consumer.auditTarget:topic}
# The consumer audit topic name if the auditTarget is topic
auditTopic: ${kafka-consumer.auditTopic:sidecar-audit}
```

The default auditTarget is a Kafka topic, and the topic name is defined in the auditTopic. It is the recommended audit target; however, it requires resource allocation and impacts the overall throughput a little bit. 

For some users who want to use a log file and then inject it into a centralized logging system like ELK, we also provide a helper to write the audit to sidecar-audit.log in the log directory. To enable it, change the auditTarget to `logfile` from `topic` and update the logback.xml to add a logger. 

```
    <!--sidecar audit log-->
    <appender name="sidecar" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/sidecar-audit.log</file> <!-- logfile location -->
        <encoder>
            <pattern>%date{ISO8601} %msg%n</pattern> <!-- the layout pattern used to format log entries -->
            <immediateFlush>true</immediateFlush>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>log/sidecar-audit.log.%i.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>5</maxIndex> <!-- max number of archived logs that are kept -->
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>200MB
            </maxFileSize> <!-- The size of the logfile that triggers a switch to a new logfile, and the current one archived -->
        </triggeringPolicy>
    </appender>

    <logger name="Sidecar" level="info" additivity="false">
        <appender-ref ref="sidecar"/>
    </logger>

```

Here are some audit records captured during the test in the sidecar-audit.json file. 

```
2021-06-18 18:11:19,516 {"id":"a2bd0e2a-95e9-41f6-a17c-1bc045463973","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"PRODUCER","topic":"test1","partition":0,"offset":496453,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"SUCCESS","stacktrace":null}
2021-06-18 18:11:19,517 {"id":"e7fe9a9f-d438-4090-8dbb-9cafdfb89008","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"PRODUCER","topic":"test1","partition":0,"offset":496454,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"SUCCESS","stacktrace":null}
2021-06-18 18:11:19,517 {"id":"e05bf3f6-7314-4e62-9b0b-5670c5c413af","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"PRODUCER","topic":"test1","partition":0,"offset":496455,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"SUCCESS","stacktrace":null}
2021-06-18 18:11:20,581 {"id":"4c8334cb-332a-4d5e-b542-f0bc6b609c05","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"REACTIVE_CONSUMER","topic":"test1","partition":0,"offset":496453,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"SUCCESS","stacktrace":null}
2021-06-18 18:11:20,585 {"id":"d25c4b2f-0ec0-4210-8f4f-ec7937e263da","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"REACTIVE_CONSUMER","topic":"test1","partition":0,"offset":496454,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"FAILURE","stacktrace":null}
2021-06-18 18:11:20,586 {"id":"eb06230c-f882-459b-9bd7-8e49b02e6de1","serviceId":"com.networknt.kafka-sidecar-1.0.0","auditType":"REACTIVE_CONSUMER","topic":"test1","partition":0,"offset":496455,"correlationId":"GMDC3LeXSLuRV0Hz5KfVHg","traceabilityId":null,"auditStatus":"SUCCESS","stacktrace":null}

```

On the producer side, we need to record all the success and failure records to the sidecar-audit file or topic through the sidecar as well. Due to threading issue, we cannot produce the audit entry to Kafka in the callback of the original message, we have to use a queue to save all the audit entries and send them after sending the original messages in batch. 

Here is the section in kafka-producer.yml file about audit.

```
# Indicator if the audit is enabled.
auditEnabled: ${kafka-producer.auditEnabled:true}
# Audit log destination topic or logfile. Default to topic
auditTarget: ${kafka-producer.auditTarget:topic}
# The consumer audit topic name if the auditTarget is topic
auditTopic: ${kafka-producer.auditTopic:sidecar-audit}

```

When auditTarget is set to `logfile` the logback.xml and the example audit entries are shown above.

To ensure the centralized processing in the future, we need to have only one audit topic across the entire organization if audit topic is used. The data in the audit topic contains only the meta data instead of the original message value. 

The retention.ms for the audit topic should be set based on the retention policy of the organization. 

### Avro to Json Transformer

When using the confluent Avro schema for producer serialization, a data type is wrapping up the data value in the JSON format. For example, the JSON format looks like this. 

```
count: {
  "int": 0
}
```
This format is hard for the consumer to convert the value to a POJO with Jackson ObjectMapper. To make the consumer side easier in a light-4j service or a backend API behind the light-mesh/kafka-sidecar, we want to convert the value to something like this. 

```
{count: 0}
```
In the kafka-consumer.yml, we have added a flag useNoWrappingAvro to control which converter to use. When no wrapping converter is used, the type will be removed. 

This flag defaults to false in the kafka-consumer.yml, and you only need to set it to true if you are using a third-party producer with the default Confluent Avro serializer. When using the kafka-sidecar producer, the type wrapper is removed in the producer record serialization. 



