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

The replay will be an endpoint exposed on the sidecar, and it can be accessed from the Control Pane like other admin endpoints. The replay will always start from the offset 0; however, the user can choose the last offset to replay. If there is no last offset specified in the request, it will replay all messages in the dead letter topic. 

Some dead letter messages still cannot be handled during the replay process, and they will be pushed to the dead letter topic again. The replay process won't process the newly produced dead-letter messages as their offset is greater than the offset specified for the request. 

Once the replay is completed, the sidecar will change the retention.ms to the last offset replayed + 1 ms to drop all the replayed messages.  

Questions:

1. Do we need to have the same number of partitions for the dead letter topic? If yes, do we need to push the replay messages to the right instance for the partition matching the original topic? 

2. If the above question is answered yes, we need to use the original message key as the dead letter topic key and design the same schema as the original topic key. 

3. The only scenario that we need to make sure that the backend API pin to a particular partition is due to the physical separation. For example, one instance is processing North American claims, and the other is processing Asian claims. 

### Audit

Audit on the sidecar is an important cross-cutting concerns that we need to address. Every records we send to the backend API and the response from the backend API should be persisted in a Kafka topic and potentially push to a SQL or No-SQL database for query and reporting. We can also use streams processing on the Control Pane to project the audit info to different dementions for query and report from the Control Pane. 

On the producer side, we need to record all the success and failure records to the sidecar-audit topic through the sidecar as well. 



Question: 

* What needs to go to the audit topic? The real records or just topic, partition, offset, key?
* How long we need to keep these audit records? 
* If we have the audit avaiable, do we still need to send the partition and offset to the backend API? 



