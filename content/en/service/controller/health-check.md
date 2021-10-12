---
title: "Health Check"
date: 2021-10-07T09:00:59-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Once a service is registered to the controller in the service post endpoint, the controller will allocate a task to send health check requests to the service instance periodically. Depending on the running mode, the health check task will be executed differently. 

### Demo Mode


### Cluster Mode


##### Register

In this mode, once a service is registered, a TaskDefinition with action 'INSERT' will be sent to light-scheduler Kafka topic directly as the controller has Kafka access and also a Kafka streams application. 


The light-schedule streams will process the task definition and push a task execution command to the target topic specified in the task definition. 

We have a health check streams application in the controller to process the task execution commands in the topic and execute the health check tasks in separate threads. The execution commands are in different Kafka partitions; they can be easily scaled across multiple controller instances. If one instance is down, the partition will be taken by another instance, and the health check commands in that partition will be taken care of by another instance. 


##### De-register


Once the registered service shuts down, it will invoke the service delete endpoint. This handle will send the same task definition with action 'DELETE' to the light-scheduler topic. The light-scheduler will process it to stop the task scheduling. 


##### Streams Processor

In the process method, we need to cast the value to the TaskDefinition object. Otherwise, we will log an error and ignore the message if the type is not correct.

* When to skip the TaskDefinition command?

When a service is de-register itself, the ServiceDeleteHandler sends a scheduler TaskDefinition to the light-scheduler to stop the health check execution. However, the last TaskDefinition object will be sent as a health check command, and it doesn't have any data object in it. It will cause NPE in the health check streams. To avoid the NPE, we skip the message by checking the action is not DELETE.

The other case that we need to skip the command is the message start timestamp is too old. First, we calculate the grace period to be less than two times of frequency. If the start time is older than the grace period from the current time, the streams will ignore the command. 


```
    long gracePeriod = TimeUtil.oneTimeUnitMillisecond(TimeUnit.valueOf(taskDefinition.getFrequency().getTimeUnit().name())) * taskDefinition.getFrequency().getTime() * 2;
    if(logger.isTraceEnabled()) logger.trace("current = " + System.currentTimeMillis() + " task start = " + taskDefinition.getStart() + " gracePeriod = " + gracePeriod);
    if(DefinitionAction.DELETE != taskDefinition.getAction() && System.currentTimeMillis() - taskDefinition.getStart() < gracePeriod) {

```

* How do we know if HTTP check or TLS check is used?

When a TaskDefinition is received from the light-scheduler, it has a data section that contains detailed information for the health check. 

If the healthPath is not empty, then it is an HTTP check. Otherwise, the health check is TTL.


* How the service removed from registry and health check scheduling is stopped?

If an HTTP or TTL check is failed, the streams will invoke the removeNode method to check if we should remove the registry and stop the health check scheduling.

```

    if(System.currentTimeMillis() - Long.valueOf(healthMap.get("deregisterCriticalServiceAfter")) > Long.valueOf(healthMap.get("lastFailedTimestamp"))) {

```

