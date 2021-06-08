---
title: "Kafka Sidecar"
date: 2021-03-16T14:25:35-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The Kafka sidecar is designed to address the following concerns for distributed microservices to leverage asynchronous event-based communications instead of synchronous request/response over HTTP. 

* Hide the complicity of Kafka client

Unlike traditional server-heavy messaging systems, Kafka's server is just a set of appended logs; however, the Kafka client is much more complicated. It is not an easy task to build a producer or consumer in microservices within a multi-threaded context. 

* Address asynchronous cross-cutting concerns

The sidecar will have middleware handlers in the request/response chain to address cross-cutting concerns, just like the light-proxy sidecar for the backend APIs. For example, the correlationId or tracerId can be injected into the message headers with the sidecar producer to propagate to downstream APIs with the sidecar consumer. 

* Easy to upgrade for new features and security vulnerabilities

There are many dependencies for Kafka and Confluent components on the client-side, and there are upgraded frequently to match the server-side version upgrade. Instead of updating all microservices to leverage the new server-side features or resolve security vulnerabilities, we can upgrade the sidecar and deploy it. 

* Support multiple languages and frameworks for APIs

Kafka client is complicated, and only the Java version is in sync with the server upgrades. The other best version that Confluent Inc. supports is the C++ version, and a lot of languages depend on it with a wrapper. There are several open-source clients for some languages, but they are always behind the server upgrades with minimum supports. With the sidecar, developers can choose any language or framework to build their APIs.
 
The goal is to allow the developers to focus on their APIs' business logic instead of the specific Kafka client API and cross-cutting concerns.

## Functional Component

### Kafka Producer

[Kafka Producer][] has an endpoint that allows backend API/APP to invoke to produce messages to one or more Kafka topics. 

### Active Consumer

[Kafka Active Consumer] has some endpoints to allow backend API/APP to create an instance, subscribe and retrieve records actively. When using this consumer, the backend API/APP has complete control of consuming the Kafka topic(s).

### Reactive Consumer

[Kafka Reactive Consumer][] will automatically poll the Kafka topic(s) on the sidecar and invoke an API endpoint on the back-end API/APP once messages are available based on the configuration. It has several endpoints to allow the light-controller to seek the offset if necessary. 

### KsqlDB Consumer

[Kafka KsqlDB Consumer][] will automatically subscribe from the KsqlDB server with a query during the server startup and callback to the backend API/App when records are received. 

### Kafka Streams

[Kafka Streams][] allows the users to develop and deploy the stream processor on the Kafka sidecar with the backend API exposes endpoints to receive the notification or the processed result. This component is working in progress. 

## Control Pane Administration

When using the Kafka Producer and Active Consumer, the backend API has full control over the producer and consumer; however, other consumers and streams are working reactively on the sidecar. The backend API will passively receive the post requests from the sidecar. If we want to update the consumer states for each consumer instance, we can leverage the control pane to access the sidecar's admin API. For example, if we want to replay events from a specific offset. For more info, please visit [Kafka Sidecar Admin][]

## Cross-cutting Concerns

One of the main goals for the Kafka sidecar is to address [cross-cutting concerns][] for asynchronous events consistently. The following non-functional concerns are address in the sidecar. 

* HTTP Request validation with OpenAPI 3.0 specification and schema validation and serialization with Confluent schema registry. 
* Propagate light-4j correlationId/traceabilityId or open tracing tracerId to the downstream service through Kafka record headers. 
* Inject correlationId/traceabilityId/tracerId to the MDC for logging statements and update logging level for sidecar packages from the control pane during runtime.
* Metrics collection on both client and service perspectives and integration with InfluxDB or enterprise metrics tools.
* The sidecar connects to the Kafka with authentication and authorization over TLS and only allow the connection from the same pod with localhost for HTTP requests.
* Sidecar auditing activities are sent to a Kafka topic through the producer and subsequently can be streams to a database for query with Confluent connect.
* Dead letter queue to handle the situation that the backend API cannot process the corrupted record successfully due to runtime exceptions. The problematic records will be pushed to a dead letter topic for future processing once the backend API is fixed and redeployed. 


## Specification

Here is the sidecar specification, and the latest version can be found [here](https://github.com/networknt/model-config/blob/master/rest/kafka-sidecar/openapi.yaml).

```
openapi: 3.0.0

info:
  title: Light Mesh Kafka Sidecar
  version: 1.0.0
  description: |-
    # Kafka producer and consumer endpoints
servers:
  - url: https://kafka.networknt.com

paths:
  '/producers/{topic}':
    post:
      operationId: postData
      summary: Post a list of users to Kafka
      parameters:
        - name: topic
          in: path
          required: true
          description: The kafka topic name
          schema:
            type: string
      requestBody:
        description: message data
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProduceRequest'
      responses:
        '200':
          description: Data successfully produced.
        '400':
          description: Unexpected error
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w

  '/consumers/{group}':
    post:
      operationId: createConsumerInstanceInGroup
      summary: Create a new consumer instance in the consumer group
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group
          schema:
            type: string
      requestBody:
        description: consumer instance request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateConsumerInstanceRequest'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CreateConsumerInstanceResponse'
        '400':
          description: Unexpected error
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w
  '/consumers/{group}/instances/{instance}':
    delete:
      operationId: deleteConsumerInstance
      summary: Delete the consumer instance
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      responses:
        '204':
          description: No Content
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w
  '/consumers/{group}/instances/{instance}/offsets':
    post:
      operationId: commmitConsumerOffsets
      summary: Commit a list of offsets for the consumer
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: topic partition offset metadata
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerOffsetCommitRequest'
      responses:
        '200':
          description: No Content
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w
    put:
      summary: Get the last committed offsets for the given partition
      operationId: getCommittedOffsets
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer committed request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerCommittedRequest'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConsumerCommittedResponse'
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w
  '/consumers/{group}/instances/{instance}/subscriptions':
    post:
      operationId: subscribeTopics
      summary: Subscribe to the given list of topics or a topic pattern to get dynamically assigned partitions.
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer seek to request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerSubscriptionRecord'
      responses:
        '204':
          description: Successful response
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w

    get:
      operationId: getSubscribedTopics
      summary: Get the current subscribed list of topics
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
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
                  $ref: '#/components/schemas/ConsumerSubscriptionResponse'
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:r

    delete:
      operationId: unsubscribeTopics
      summary: Unsubscribe from topics currently subscribed
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      responses:
        '204':
          description: No Content
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w


  '/consumers/{group}/instances/{instance}/assignments':
    post:
      operationId: assignPartitions
      summary: Manually assign a list of partitions to this consumer.
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer seek to request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerAssignmentRequest'
      responses:
        '204':
          description: Successful response
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w

    get:
      operationId: getAssignedPartitions
      summary: Get the list of partitions currently manually assigned to this consumer
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
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
                  $ref: '#/components/schemas/ConsumerAssignmentResponse'
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:r


  '/consumers/{group}/instances/{instance}/positions':
    post:
      operationId: overrideFetchOffset
      summary: Overrides the fetch offsets that the consumer will use for the next set of records to fetch
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer seek to request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerSeekRequest'
      responses:
        '204':
          description: Successful response
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w


  '/consumers/{group}/instances/{instance}/positions/first':
    post:
      operationId: seekFirstOffset
      summary: Seek to the first offset for each of the given partitions
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer seek to request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerSeekToRequest'
      responses:
        '204':
          description: Successful response
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w

  '/consumers/{group}/instances/{instance}/positions/last':
    post:
      operationId: seekLastOffset
      summary: Seek to the last offset for each of the given partitions
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
      requestBody:
        description: consumer seek to request
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConsumerSeekToRequest'
      responses:
        '204':
          description: Successful response
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w
  '/consumers/{group}/instances/{instance}/records':
    get:
      operationId: fetchTopicData
      summary: Fetch data for the topics or partitions specified using one of the subscribe/assign APIs
      parameters:
        - name: group
          in: path
          required: true
          description: The consumer group name
          schema:
            type: string
        - name: instance
          in: path
          required: true
          description: The Id of the consumer instance
          schema:
            type: string
        - name: timeout
          in: query
          description: Maximum amount of milliseoncds the sidecar will spend fetching records
          required: false
          schema:
            type: integer
        - name: max_bytes
          in: query
          description: Maximum number of bytes of the unencoded keys and values in the response
          required: false
          schema:
            type: integer
        - name: format
          in: query
          description: The format of response
          required: true
          schema:
            type: string
            enum:
              - binary
              - json
              - avro
              - jsonschema
              - protobuf

      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/ConsumerRecord'
        '404':
          description: Consumer instance not found
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:r

components:
  securitySchemes:
    kafka_auth:
      type: oauth2
      description: This API uses OAuth 2.0 with the client credential grant flow.
      flows:
        clientCredentials:
          tokenUrl: 'https://localhost:6882/token'
          scopes:
            kafka:w: Kafka producer
            kafka:r: Kafka consumer
  schemas:
    ProduceRequest:
      type: object
      properties:
        keyFormat:
          type: integer
          enum: [
            0,
            1,
            2,
            3,
            4
          ]
        keySchema:
          type: string
        keySchemaId:
          type: integer
        keySchemaVersion:
          type: integer
        keySchemaSubject:
          type: string
        valueFormat:
          type: integer
          enum: [
            0,
            1,
            2,
            3,
            4
          ]
        valueSchema:
          type: string
        valueSchemaId:
          type: integer
        valueSchemaVersion:
          type: integer
        valueSchemaSubject:
          type: string
        records:
          type: array
          items:
            type: object
      required:
        - records
    CreateConsumerInstanceRequest:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        format:
          type: string
          enum:
            - binary
            - avro
            - json
            - jsonschema
            - protobuf
        autoOffsetReset:
          type: string
        autoCommitEnable:
          type: string
        responseMinBytes:
          type: integer
          minimum: -1
          maximum: 10000000
        requestWaitMs:
          type: integer
    CreateConsumerInstanceResponse:
      type: object
      properties:
        instanceId:
          type: string
        baseUri:
          type: string
    ConsumerOffsetCommitRequest:
      type: object
      properties:
        offsets:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartitionOffsetMetadata"
    TopicPartitionOffsetMetadata:
      type: object
      properties:
        topic:
          type: string
        partition:
          type: integer
          minimum: 0
        offset:
          type: number
        metadata:
          type: string

    ConsumerCommittedResponse:
      type: object
      properties:
        offsets:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartitionOffsetMetadata"
    ConsumerCommittedRequest:
      type: object
      properties:
        offsets:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartition"

    TopicPartition:
      type: object
      properties:
        topic:
          type: string
        partition:
          type: integer
          minimum: 0

    ConsumerRecord:
      type: object
      properties:
        topic:
          type: string
        key:
          type: object
        value:
          type: object
        partition:
          type: integer
        offset:
          type: number
    ConsumerSeekToRequest:
      type: object
      properties:
        partitions:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartition"
    ConsumerSeekRequest:
      type: object
      properties:
        offsets:
          type: array
          items:
            "$ref": "#/components/schemas/PartitionOffset"
        timestamps:
          type: array
          items:
            "$ref": "#/components/schemas/PartitionTimestamp"

    PartitionOffset:
      type: object
      properties:
        topic:
          type: string
        partition:
          type: integer
        offset:
          type: number
        metadata:
          type: string

    PartitionTimestamp:
      type: object
      properties:
        topic:
          type: string
        partition:
          type: integer
        timestamp:
          type: string
        metadata:
          type: string

    ConsumerAssignmentRequest:
      type: object
      properties:
        partitions:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartition"
    ConsumerAssignmentResponse:
      type: object
      properties:
        partitions:
          type: array
          items:
            "$ref": "#/components/schemas/TopicPartition"

    ConsumerSubscriptionRecord:
      type: object
      properties:
        topics:
          type: array
          items:
            type: string
        topic_pattern:
          type: string

    ConsumerSubscriptionResponse:
      type: object
      properties:
        topics:
          type: array
          items:
            type: string

    Status:
      type: object
      properties:
        statusCode:
          description: HTTP response code
          type: integer
        code:
          description: Code is the machine-readable error code
          type: string
        message:
          description: Error messsage
          type: string
        description:
          description: The detailed description of the error status
          type: string
      required:
        - statusCode
        - code

```


## Deployment

When deploying the Kafka Sidecar to a Kubernetes cluster, we need to allocate more memory for the container because Kafka uses more memory when producing and consuming records. 

```
resources:
  limits:
    cpu: 500m
    memory: 1024Mi
  requests:
    cpu: 250m
    memory: 768Mi  
```

Also, set the Java command line memory options as an environment variable or in the command line in Dockerfile.

```
env:
  -name: JVM_OPTS
  value: -Xmx768m -Xms960m
```

## Performance

When using the Kafka sidecar to interact with Kafka Cluster, the Kafka Client will be wrapped by the sidecar. The backend API will not have any dependencies on Kafka libraries. As we know, the Kafka client is CPU and memory heavy due to the cache. 

### Producer
For the producer, we need to control the size of the request body to be smaller than 1MB. If you have too many messages to be sent to Kafka, divide them into smaller sizes and send multiple requests. 

Depending on the size of the message, we recmmend sending hundreds or thousands of messages each batch, and the total size is smaller than 1MB. 

If you have a smaller message size, the batch size should be smaller as the number of messages in the batch will impact the throughput dramatically. With more messages in the batch, Kafka Sidecar has to divide them into several Kafka batches and write the audit in the same handler. It requires more memory and CPU power. Also, the Jackson parser is getting slower when there are too many small objects in a JSON array than fewer bigger objects in a JSON array when both JSON are about the same size. 

If you have some special case that one message that is bigger than 1MB (should be very rare), then you need to update the server.yml to all light-4j to accept requests bigger than 1MB. 

```
# Set the max transfer file size for uploading files—default to 1000000, which is 1 MB.
# maxTransferFileSize: ${server.maxTransferFileSize:1000000}
```

### Reactive Consumer

For each reactive consumer application, we need to carefully design the batch size to reach the optimal performance in terms of throughput and CPU/memory usage. 

The bigger the batch size, the better throughput to consume more messages in a short time. However, more memory and more CPU power 
will be utilized when the batch size is increased. 

When tuning the performance for the Reactive Consumer, we need to consider the following factors. 

##### Batch Size

It is configurable in the kafka-consumer.yml, and the default value is 100K. With an average record size of 1KB, a batch will contain about 100 messages. 

```
# maximum number of bytes message keys and values returned. Default to 100*1024
requestMaxBytes: ${kafka-consumer.requestMaxBytes:102400}
```

##### Record Size

When the backend is consuming the messages, each message will take some time. The more messages in the batch, the more time is needed to process the entire batch. If the record size is smaller, please make sure the batch size is smaller so that the backend won't be overloaded. In general, we should make sure each batch only container about 100 records.

##### Backend Processing Time

How long will it take for the backend to process each record will impact the performance and the design for the batch size. Depending on the business logic, we need to ensure the entire processing time to be limited to 20 to 30 seconds. The average processing time should be used to design the batch size along with the record size. 

##### fetch.max.bytes

Once we decided on the requestMaxBytes, we need to adjust several Kafka properties to make sure that the polling from Kafka is more efficient. As we don't want to cache more data on the sidecar, we want to make sure that the fetch.max.bytes are just about the requestMaxBytes. 

```
  # max fetch size from Kafka cluster. Default 50mb is too big for cache consumption on the sidecar
  fetch.max.bytes: ${kafka-consumer.fetch.max.bytes:102400}

```

##### max.poll.records

This Kafka property gives us more control over the Kafka consumer. The default value is 100, and it should be adjusted based on the record size and the batch size. 

```
  # max pol records default is 500. Adjust it based on the size of the records to make sure each poll
  # is similar to requestMaxBytes down below.
  max.poll.records: ${kafka-consumer.max.poll.records:100}

```

##### max.partition.fetch.bytes

This property is similar to the` fetch.max.bytes` but it is defined on the partition level. The value should be the same as the `fetch.max.bytes`, which in line with the requestMaxBytes. 

```
  # The maximum amount of data per-partition the server will return. Records are fetched in batches by the consumer.
  # If the first record batch in the first non-empty partition of the fetch is larger than this limit, the batch will still be returned to ensure that the consumer can make progress.
  max.partition.fetch.bytes: ${kafka-consumer.max.partition.fetch.bytes:102400}
```

[Kafka Active Consumer]: /service/mesh/kafka/active-consumer/
[Kafka Reactive Consumer]: /service/mesh/kafka/reactive-consumer/
[Kafka Producer]: /service/mesh/kafka/producer/
[Kafka KsqlDB Consumer]: /service/mesh/kafka/ksqldb-consumer/
[Kafka Streams]: /service/mesh/kafka/kafka-streams/
[Kafka Sidecar Admin]: /service/mesh/kafka/sidecar-amdin/
[cross-cutting concerns]: /service/mesh/kafka/sidecar-ccc/
