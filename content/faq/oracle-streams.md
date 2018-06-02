---
title: "Oracle Streams"
date: 2018-06-01T21:38:09-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


### Issue

We were asked from one of our customers who has unlimited Oracle license if our messaging based framework can utilize Oracle Streams instead of GoldenGate which requires a separate license.

After several days of investigation,  we don't think Oracle Streams is working in this use case.




### Oracle Stream


The Oracle Steams is not fit our CDC solution. It can capture the data change to the queue, but the queue is oracle database queue.


Oracle Steams use Oracle ANYDATA queue for message store.

The easiest way to create an ANYDATA queue is to use the SET_UP_QUEUE procedure in the DBMS_STREAMS_ADM package.
This procedure enables you to specify the following settings for the ANYDATAqueue it creates:


	•	The queue table for the queue 

	•	A storage clause for the queue table 

	•	The queue name 

	•	A queue user that will be configured as a secure queue user of the queue and granted ENQUEUE and DEQUEUE privileges on the queue 

	•	A comment for the queue 


For example, run the following procedure to create an ANYDATA queue with the SET_UP_QUEUE procedure:

```

BEGIN
  DBMS_STREAMS_ADM.SET_UP_QUEUE(
    queue_table => 'strmadmin.streams_queue_table',
    queue_name  => 'strmadmin.streams_queue',
    queue_user  => 'hr');
END;
/
```



And if we want to use other messaging systems as message broker for CDC, we need to use Oracle Message Gateway. Messaging Gateway enables communication between applications based on non-Oracle messaging systems and Oracle Streams AQ.

But the Oracle Message Gateway currently only supports the integration of Oracle Streams AQ with applications based on WebSphere MQ 6.0 and TIB/Rendezvous 7.2.

it doesn't support kafka (framework default message broker for CDC) for now


### Other solutions:


1. If we simply want to read (select query) and push to Kafka, simple JDBC code is enough; Our frameworks ([light-eventuate-4j][] / [light-tram-4j][]) has provided the Oracle DB pulling CDC solution.



2. To move change data in real-time from Oracle transactional databases to Kafka you need to first use a Change Data Capture (CDC) proprietary tool which requires purchasing a commercial license such as Oracle’s Golden Gate,
Attunity Replicate, Dbvisit Replicate or Striim. Then, you can leverage the Kafka Connect connectors that they all provide.


please refer to [products list][];



3. Debezium, an open source CDC tool from Redhat, is planning to work on a connector that is not relying on Oracle Golden Gate license.

[JIRA][] is here;





[light-eventuate-4j]: https://github.com/networknt/light-eventuate-4j
[light-tram-4j]: https://github.com/networknt/light-tram-4j
[products list]: https://www.confluent.io/product/connectors/
[JIRA]: https://issues.jboss.org/browse/DBZ-137?_sscc=t

