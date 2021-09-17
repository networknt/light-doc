---
title: "Light Scheduler"
date: 2021-09-16T09:01:12-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In any enterprise with distributed cloud applications, task scheduling is a key requirement. A high available and fault-tolerant task scheduler will help to build robust cloud applications to meet business goals.

Traditional task schedulers like Quartz using a database for task definition store, and it is hard to scale the services horizontally. By using Kafka, Kafka Streams and State Store, the scheduler can be scaled easily. 


### Design

Task definition commands will be pushed to input topic (light-scheduler by default) with multiple partitions when a REST endpoint is called. In the Avro formatted definition command, there are three possible actions: INSERT, UPDATE and DELETE to create, update or delete a task definition in the State store backed by Kafka changelog topics per time unit.

A streaming app will take the command from the input topic and, based on the action to add/update/delete the task definition to/from the State Store that is backed by Kafka changelog topics per time unit.


##### Processing Flow.

* Task definitions will be pushed to the input topic light-scheduler with INSERT/UPDATE/DELETE action by the API endpoint /schedulers post request.

* The light-scheduler instance with Kafka Streams processor will read these definitions from the input topic partitions and insert/update/delete the entity to/from the respective state stores.

* The light-scheduler will expose API endpoints to retrieve task definitions from the state store with interactive queries. 

* The Transformer implementation will call a Punctuator periodically per TimeUnit.

* The Punctuator reads all the Task definitions from the store and sends them out for execution based on the topic defined in the task definition. 

* Five schedulers are created in the transformer: MILLISECONDS, SECONDS, MINUTES, HOURS, and DAYS. And the punctuator will be called at each TimeUnit. The exact time to send out the task is based on the start timestamp and the frequency (TimeUnit and Time) in the task definition. 

* The scheduled tasks will be pushed to each topic specified in the task definition, and target applications will use Kafka streams to process the tasks in the streams processor. 

* If one light-scheduler instance is shut down, other instances will take over the partitions that were owned by the shutdown instance.



##### Avro Schemas

The Avro schemas are defined in the [light-portal][] repository. 


TaskDefinitionKey.avsc

```
{
  "namespace": "com.networknt.scheduler",
  "type": "record",
  "name": "TaskDefinitionKey",
  "fields": [
    {"name": "host","type": "string"},
    {"name": "name","type": "string"}
  ]
}

```

This is the Kafka producer record key used by all the topics, including changelog topics. The host is to support multi-tenancy for the subscribed users in the public cloud. By default, we are using networknt.com for the private instance. 

The task's name should be unique across the host—for example, serviceId + a meaningful name.


FrequencyTimeUnit.avsc

```
{
  "namespace": "com.networknt.scheduler",
  "type": "enum",
  "name": "TimeUnit",
  "symbols": ["MILLISECONDS", "SECONDS", "MINUTES", "HOURS", "DAYS"]
}

```

This is an enum mapped to Java TimeUnit. We support five schedulers per each TimeUnit above. 


TaskFrequency.avsc

```
{
  "namespace": "com.networknt.scheduler",
  "type": "record",
  "name": "TaskFrequency",
  "fields": [
    {"name": "timeUnit", "type": "TimeUnit"},
    {"name": "time", "type": "int"}
  ]
}
```

This is the frequency for the task definition that contains a TimeUnit and an integer time field. If you want to send out a scheduled task every 10 seconds, you can set the time unit to SECONDS and the time to 10. If you want to send out a scheduled task every two minutes, then set the time unit to MINUTES and time to 2. 


TaskDefinitionAction.avsc

```
{
  "namespace": "com.networknt.scheduler",
  "type": "enum",
  "name": "DefinitionAction",
  "symbols": ["INSERT", "UPDATE", "DELETE"]
}

```

This is an enum to list all the actions you can do with the task definition command. The INSERT action is to create a brand new task definition and start the task scheduling. The UPDATE action is to update the existing task definition with the same key. The DELETE action is to stop the task scheduling. In most cases, we only need INSERT and DELETE to create and cancel the task. 


TaskDefinition.avsc

```
{
  "namespace": "com.networknt.scheduler",
  "type": "record",
  "name": "TaskDefinition",
  "fields": [
    {"name": "host","type": "string"},
    {"name": "name","type": "string"},
    {"name": "action","type": "DefinitionAction"},
    {"name": "frequency","type": ["null", "TaskFrequency"], "default": null},
    {"name": "topic", "type": "string"},
    {"name": "start", "type": ["null", "long"], "default": null},
    {"name": "data","type": [ "null",
      {
        "type": "map",
        "values": "string"
      }
    ], "default": null}
  ]
}
```

The host and name are part of the key, and the combination is unique. The topic is the output topic for the scheduled task. The start timestamp is optional, and you only need to populate it if you want to schedule a task starting at a future time. If it doesn't exist in the request, then the current time converted to the next exact TimeUnit will be used. 

The data field is an open structure so that users can put some extra information to assist the task execution. It is optional, but most cases will be populated based on the task itself. 


##### Endpoints

The full OpenAPI specification can be found at [model-config](https://github.com/networknt/model-config/tree/master/rest/light-scheduler) along with the codegen config and README.md. 


Here are the two endpoints in the specification.

```

  /schedulers:
    post:
      operationId: sendScheduleTask
      summary: Send a new schedule task definition to input topic
      requestBody:
        description: task definition
        required: true
        content:
          application/json:
            schema: 
              $ref: '#/components/schemas/TaskDefinition'
      responses:
        '200':
          description: Schedule task definition sent.
        '400':
          description: Unexpected error
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - scheduler_auth:
            - sched:w

    get:
      operationId: getScheduleTaskDefinitions
      summary: Get a list of schedule task definitions.
      parameters:
        - name: host
          in: query
          required: true
          description: Task definition host
          schema:
            type: string
        - name: name
          in: query
          required: true
          description: Task definition name
          schema:
            type: string
        - name: unit
          in: query
          required: true
          description: Task definition time unit
          schema:
            type: string
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/TaskDefinition'
        '400':
          description: Unexpected error
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - scheduler_auth:
            - sched:r
```

### References

* Tutorial can be found at [here](/tutorial/scheduler/).
* The repository can be found at [light-scheduler][] in networknt organization on GitHub.


[light-portal]: https://github.com/lightapi/light-portal/tree/master/portal-event/src/main/resources/schemas

[light-scheduler]: https://github.com/networknt/light-scheduler
