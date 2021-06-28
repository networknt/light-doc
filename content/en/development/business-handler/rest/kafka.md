---
title: "Rest with Kafka Access"
date: 2020-08-30T23:38:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We used to leverage database for Event Sourcing and CQRS with light-eventuate-4, light-tram-4j and light-saga-4j. Due to multiple dependencies, it is hard to use. With the latest Kafka supports transactions, we have developed Light-Kafka to use Kafka Streams for Event Sourcing and CQRS. 

When using light-kafka with Event Sourcing, we don't put producer endpoints and consumer endpoints in the same service. For the rest of the document, we will call the producer as the command service and consumer as the query service. 

### Command Service

The command service is the producer of the Kafka streams. It exposes REST endpoints to the outside to accept data, validate and push to a Kafka topic. The format of the data normally will be JSON or AVRO. JSON is acceptable; however, we highly recommend AVRO for any enterprise application if backward and forward compatibility is required. 

To ensure efficiency in publishing events to a Kafka topic, we have created an in-memory queue to cache the data for a short period and push them to Kafka all together. 

Here is an example of pushing a message. 

```
        // get nonce for the transaction
        Result<String> result = HybridQueryClient.getNonceByEmail(exchange, userId);
        if(result.isFailure()) {
            return NioUtils.toByteBuffer(getStatus(exchange, result.getError()));
        }
        Long nonce = Long.valueOf(result.getResult());

        EventId eventId = EventId.newBuilder()
                .setId(userId)
                .setNonce(nonce)
                .build();
        MarketClientCreatedEvent event = MarketClientCreatedEvent.newBuilder()
                .setEventId(eventId)
                .setKeyId(0) // keyId is clientId
                .setHost((String)map.get("host"))
                .setClientId(clientId)
                .setValue(JsonMapper.toJson(map))
                .setTimestamp(System.currentTimeMillis())
                .build();

        AvroSerializer serializer = new AvroSerializer();
        byte[] bytes = serializer.serialize(event);

        ProducerRecord<byte[], byte[]> record = new ProducerRecord<>(config.getTopic(), clientId.getBytes(StandardCharsets.UTF_8), bytes);
        LightProducer producer = SingletonServiceFactory.getBean(LightProducer.class);
        BlockingQueue<ProducerRecord<byte[], byte[]>> txQueue = producer.getTxQueue();
        try {
            txQueue.put(record);
        } catch (InterruptedException e) {
            logger.error("Exception:", e);
            return NioUtils.toByteBuffer(getStatus(exchange, SEND_MESSAGE_EXCEPITON, e.getMessage(), userId));
        }

```

For more examples, please refer to [maproot command](https://github.com/networknt/maproot/tree/master/covid-command) service.

### Query Service

For the query service, it exposes REST endpoints for users to query the data in different projections. For example, one endpoint might get an entity with the key, and another endpoint might get a list of entities for admin purposes. The events persisted in the Kafka topic can also be projected later to go back to the past. For example, to generate a monthly new registered user report from day one. 

Using Kafka streams, we can project the events to the built-in key/value store in the streams processor. So for each query service, we must add a streams processor. Please find an [example](https://github.com/networknt/maproot/blob/master/covid-query/src/main/java/net/lightapi/portal/covid/query/CovidQueryStreams.java) in the maproot repository. 

With the projected local key/value stores, the query endpoint will just query the local store's data. Here is an example.

```
        Map<String, String> map = (Map<String, String>)input;
        String email = map.get("email");
        // email shouldn't be null as the schema validation is done.
        ReadOnlyKeyValueStore<String, Long> keyValueStore = UserQueryStartup.nonceStreams.getUserNonceStore();
        Long data = keyValueStore.get(email);
        if(data != null) {
            return NioUtils.toByteBuffer(data.toString());
        } else {
            // lookup other instance
            StreamsMetadata metadata = UserQueryStartup.nonceStreams.getUserNonceStreamsMetadata(email);
            if(logger.isDebugEnabled()) logger.debug("found address in another instance " + metadata.host() + ":" + metadata.port());
            // Don't call if the host and port is the same as the current instance. Dead loop will occur.
            String url = "https://" + metadata.host() + ":" + metadata.port();
            if(NetUtils.getLocalAddressByDatagram().equals(metadata.host()) && Server.getServerConfig().getHttpsPort() == metadata.port()) {
                // TODO remove this block if we never seen the following error.
                logger.error("******Kafka returns the same instance!");
                return NioUtils.toByteBuffer(getStatus(exchange, USER_NOT_FOUND_BY_EMAIL, email));
            } else {
                Result<String> resultNonce = HybridQueryClient.getNonceByEmail(exchange, url, email);
                if(resultNonce.isSuccess()) {
                    return NioUtils.toByteBuffer(resultNonce.getResult());
                }
            }
            return NioUtils.toByteBuffer(getStatus(exchange, USER_NOT_FOUND_BY_EMAIL, email));
        }

```

Please note that the entity might be on another node in the cluster, and you need to query Kafka metadata to find out which node to go to get the data. 

For more examples, please refer to [maproot query service](https://github.com/networknt/maproot/tree/master/covid-query).


