---
title: "Kafka Streams Query"
date: 2021-01-06T09:27:14-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In the previous [producer-consumer][] tutorial, we have demonstrated how to build a producer and consumer with light-rest-4j. In this tutorial, we will create a microservice with Kafka Streams to simulate the event sourcing. The previously built producer will be reused to simulate the command side for the CQRS. 

The project is the query service that has /query and /query/{userId} get endpoints. 

### Codegen

As usual, you can find the specification, config.json and a README.md with a light-codegen command line in the model-config repository.

https://github.com/networknt/model-config/tree/master/kafka/stream-query

Follow the README.md command line, we can generate a project in the light-example-4j/kafka folder.

https://github.com/networknt/light-example-4j/tree/master/kafka/stream-query

The project can be built and started with the following command.

```
mvn clean install exec:exec
```

### pom.xml

We need to add the kafka-streams dependencies to the pom.xml

```
        <version.kafka>3.2.0</version.kafka>

        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-common</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>${version.kafka}</version>
        </dependency>
```

### UserQueryStartupHook.java

We need a startup hook to start the Kafka Streams during the server startup. 

```
package com.networknt.kafka;

import com.networknt.server.Server;
import com.networknt.server.StartupHookProvider;
import com.networknt.utility.NetUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserQueryStartupHook implements StartupHookProvider {
    static final Logger logger = LoggerFactory.getLogger(UserQueryStartupHook.class);
    public static UserQueryStreams streams = null;
    @Override
    public void onStartup() {
        int port = Server.getServerConfig().getHttpsPort();
        String ip = NetUtils.getLocalAddressByDatagram();
        logger.info("ip = " + ip + " port = " + port);
        streams = new UserQueryStreams();
        // start the kafka stream process
        streams.start(ip, port);
    }
}

```

### UserQueryShutdownHook.java

We need a shutdown hook to close the Kafka Streams during the server shutdown. 

```
package com.networknt.kafka;

import com.networknt.server.ShutdownHookProvider;

public class UserQueryShutdownHook implements ShutdownHookProvider {
    @Override
    public void onShutdown() {
        if(UserQueryStartupHook.streams != null) UserQueryStartupHook.streams.close();
    }
}

```

### service.yml

We need to put the startup and shutdown hooks to the service.yml to invoke them. 

```
# Singleton service factory configuration/IoC injection
singletons:
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.kafka.UserQueryStartupHook
  
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.kafka.UserQueryShutdownHook

```

### kafka-streams.yml

Before we start streams processor implementation, we need to create a config file. 

```
applicationId: user-query
bootstrapServers: localhost:9092
processingGuarantee: exactly_once
replicationFactor: 3
# Only set to true right after the streams reset.
# cleanUp: false
cleanUp: true
```

### UserQueryStreams.java

This is the event processor that will create the local key/value store for the handler to access once the API is called from the Restful endpoint. 


```
package com.networknt.kafka;

import com.networknt.config.Config;
import com.networknt.kafka.common.KafkaStreamsConfig;
import com.networknt.kafka.streams.LightStreams;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StoreQueryParameters;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.processor.AbstractProcessor;
import org.apache.kafka.streams.processor.ProcessorContext;
import org.apache.kafka.streams.state.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.charset.StandardCharsets;
import java.util.Properties;

public class UserQueryStreams implements LightStreams {
    static private final Logger logger = LoggerFactory.getLogger(UserQueryStreams.class);
    private static final String APP = "userQuery";

    static private Properties streamsProps;
    static final KafkaStreamsConfig config = (KafkaStreamsConfig) Config.getInstance().getJsonObjectConfig(KafkaStreamsConfig.CONFIG_NAME, KafkaStreamsConfig.class);

    static {
        streamsProps = new Properties();
        streamsProps.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, config.getBootstrapServers());
        streamsProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    }

    private static final String userId = "user-id-store"; // this is a local store from userId to user object.

    KafkaStreams userQueryStreams;

    public UserQueryStreams() {
        logger.info("UserQueryStreams is created");
    }

    public ReadOnlyKeyValueStore<String, String> getUserIdStore() {
        return userQueryStreams.store(StoreQueryParameters.fromNameAndType(userId, QueryableStoreTypes.keyValueStore()));
    }

    public StreamsMetadata getUserIdStreamsMetadata(String id) {
        return userQueryStreams.metadataForKey(userId,  id, Serdes.String().serializer());
    }

    private void startUserQueryStreams(String ip, int port) {

        StoreBuilder<KeyValueStore<String, String>> keyValueUserIdStoreBuilder =
                Stores.keyValueStoreBuilder(Stores.persistentKeyValueStore(userId),
                        Serdes.String(),
                        Serdes.String());

        final Topology topology = new Topology();
        topology.addSource("SourceTopicProcessor", "test");
        topology.addProcessor("UserEventProcessor", UserEventProcessor::new, "SourceTopicProcessor");
        topology.addStateStore(keyValueUserIdStoreBuilder, "UserEventProcessor");

        streamsProps.put(StreamsConfig.APPLICATION_ID_CONFIG, "user-query");
        streamsProps.put(StreamsConfig.APPLICATION_SERVER_CONFIG, ip + ":" + port);
        userQueryStreams = new KafkaStreams(topology, streamsProps);
        if(config.isCleanUp()) {
            userQueryStreams.cleanUp();
        }
        userQueryStreams.start();
    }

    public static class UserEventProcessor extends AbstractProcessor<byte[], byte[]> {

        private ProcessorContext pc;
        private KeyValueStore<String, String> userIdStore;

        public UserEventProcessor() {
        }

        @Override
        public void init(ProcessorContext pc) {
            this.pc = pc;
            this.userIdStore = (KeyValueStore<String, String>) pc.getStateStore(userId);
            if (logger.isInfoEnabled()) logger.info("Processor initialized");
        }

        @Override
        public void process(byte[] key, byte[] value) {
            String userId = new String(key, StandardCharsets.UTF_8);
            String userStr = new String(value, StandardCharsets.UTF_8);
            if(logger.isTraceEnabled()) logger.trace("Event = " + userStr);
            userIdStore.put(userId, userStr);
        }
    }

    @Override
    public void start(String ip, int port) {
        if(logger.isDebugEnabled()) logger.debug("userQueryStreams is starting...");
        startUserQueryStreams(ip, port);
    }

    @Override
    public void close() {
        if(logger.isDebugEnabled()) logger.debug("userQueryStreams is closing...");
        userQueryStreams.close();
    }
}

```

### QueryUserIdGetHandler.java

Now, let's update the generated handler for the endpoint. 


```
package com.networknt.kafka.handler;

import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import com.networknt.config.JsonMapper;
import com.networknt.handler.LightHttpHandler;
import com.networknt.kafka.UserQueryStartupHook;
import com.networknt.monad.Failure;
import com.networknt.monad.Result;
import com.networknt.monad.Success;
import com.networknt.server.Server;
import com.networknt.status.Status;
import com.networknt.utility.NetUtils;
import com.networknt.utility.NioUtils;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Headers;
import io.undertow.util.HttpString;
import io.undertow.util.Methods;
import org.apache.kafka.streams.state.ReadOnlyKeyValueStore;
import org.apache.kafka.streams.state.StreamsMetadata;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

/**
For more information on how to write business handlers, please check the link below.
https://doc.networknt.com/development/business-handler/rest/
*/
public class QueryUserIdGetHandler implements LightHttpHandler {
    private static final Logger logger = LoggerFactory.getLogger(QueryUserIdGetHandler.class);
    static Map<String, ClientConnection> connCache = new ConcurrentHashMap<>();
    static Http2Client client = Http2Client.getInstance();

    private static final String OBJECT_NOT_FOUND = "ERR11637";
    private static final String GENERIC_EXCEPTION = "ERR10014";

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        String userId = exchange.getQueryParameters().get("userId").getFirst();
        if(logger.isTraceEnabled()) logger.trace("userId = " + userId);

        ReadOnlyKeyValueStore<String, String> keyValueStore = UserQueryStartupHook.streams.getUserIdStore();
        String data = keyValueStore.get(userId);
        if(data != null) {
            exchange.getResponseHeaders().add(Headers.CONTENT_TYPE, "application/json");
            exchange.getResponseSender().send(data);
        } else {
            StreamsMetadata metadata = UserQueryStartupHook.streams.getUserIdStreamsMetadata(userId);
            if(logger.isDebugEnabled()) logger.debug("found address in another instance " + metadata.host() + ":" + metadata.port());
            String url = "https://" + metadata.host() + ":" + metadata.port();
            if(NetUtils.getLocalAddressByDatagram().equals(metadata.host()) && Server.getServerConfig().getHttpsPort() == metadata.port()) {
                setExchangeStatus(exchange, OBJECT_NOT_FOUND, "user", userId);
                return;
            } else {
                Result<String> resultStatus = getUserById(userId, exchange, url);
                if (resultStatus.isSuccess()) {
                    exchange.getResponseHeaders().add(Headers.CONTENT_TYPE, "application/json");
                    exchange.getResponseSender().send(resultStatus.getResult());
                    return;
                }
            }
            setExchangeStatus(exchange, OBJECT_NOT_FOUND, "user", userId);
            return;
        }
    }

    public static Result<String> getUserById(String userId, HttpServerExchange exchange, String url) {
        Result<String> result = null;
        try {
            ClientConnection conn = connCache.get(url);
            if(conn == null || !conn.isOpen()) {
                conn = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
                connCache.put(url, conn);
            }
            // Create one CountDownLatch that will be reset in the callback function
            final CountDownLatch latch = new CountDownLatch(1);
            // Create an AtomicReference object to receive ClientResponse from callback function
            final AtomicReference<ClientResponse> reference = new AtomicReference<>();
            String path = "/query/" + userId;
            final ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(path);
            String token = exchange.getRequestHeaders().getFirst(Headers.AUTHORIZATION);
            if(token != null) request.getRequestHeaders().put(Headers.AUTHORIZATION, token);
            request.getRequestHeaders().put(Headers.HOST, "localhost");
            conn.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
            if(statusCode != 200) {
                Status status = Config.getInstance().getMapper().readValue(body, Status.class);
                result = Failure.of(status);
            } else result = Success.of(body);
        } catch (Exception e) {
            logger.error("Exception:", e);
            Status status = new Status(GENERIC_EXCEPTION, e.getMessage());
            result = Failure.of(status);
        }
        return result;
    }

}

```

### Test

Assuming that the Kafka server is still running locally and there are some users in the test topic produced by the [producer-consumer][] example. 

```
mvn clean install exec:exec
```

Once the server is up and running, you can send a request. 

```
curl -k --location --request GET 'https://localhost:8443/query/stevehu'
```

To query another user, use the following command.

```
curl -k --location --request GET 'https://localhost:8443/query/joedoe'
```

### Start Multiple Instances

When we create the test topic, we have created three partitions. This means that we can start up to three instances of this service and each instance is responsible for one partition. The local data store will be divided by the partition as well. 

If one instance receives a request with userId 'stevehu' and the user doesn't exist in the local store, it will ask Kafka which instance contain this key and send a request to that instance to query the user data. The logic is in the QueryUserIdGetHandler.java to call another instance if necessary. 

For users who are interested, please externalize the configuration and start three instances to listen to three ports and test it. 

### Summary

This is a very simple example to demo the event sourcing and CQRS with Kafka Streams. To make it as simple as possible, I just use a json object to simulate the event and there is no header for tracing and security etc. In real project, we should use Avro or Protobuf with schema registry for backward and forward compitability. If we ever to use JSON, we need to design different event types with a header and using JSON schema to ensure the data integrity and validity. 

The same architecture is used by the light-portal and all other projects for our cloud services. The only differences are we use Avro for the events and light-hybrid-4j serverless framework instead of light-rest-4j. 

[producer-consumer]: /tutorial/light-kafka/producer-consumer/
