---
title: "Light Tram 4j"
date: 2017-12-18T13:45:57-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 10
weight: 10
sections_weight: 10
aliases: []
toc: false
draft: false
---

light-tram-4j is a port from [eventuate-tram-core][] built by Chris Richardson. The client
API is the same but implemented not in Spring framework but light-4j instead. 

There are three frameworks in light-platform focused on event driven microservices and light-tram-4j is the most fundamental one. Tram stands for transactional messaging. Ir enables an application to automatically update state and send a message or a domain event as part of a database transaction. Tram is a framework of ensuring data consistency within a microservices architecture.

light-tram-4j provides several messaging abstractions:

* messages - send and receive messages over named channels
* events - publish domain events and subscribe to domain events
* commands - asynchronoously send a command to a service and receive a reply

### How it works.

Tram messaging implements the Application events pattern. A message producer inserts 
events into an OUTBOX table as part of the ACID transaction that updates data, such as 
JPA entities. A separate message relay (a.k.a. CDC service) publishes messages to the 
message broker.

The message relay works in one of two ways:

* Transaction log tailing - currently implemented for MySQL

* Polling - for other databases such as Oracle, Postgres


### Example Application

* [Todo list][] A Todo list application, which publishes domain event using light-tram-4j


### Transactional messaging

Send a message using MessageProducer:

```java
public interface MessageProducer {
  void send(String destination, Message message);
}
```

Receive messages using:

```java
public interface MessageConsumer {
  void subscribe(String subscriberId, Set<String> channels, MessageHandler handler);
}
```

### Transactional domain events

The domain event package builds on the core APIs.

Publish domain events using the DomainEventPublisher interface:

```java
public interface DomainEventPublisher {

  void publish(String aggregateType, Object aggregateId, List<DomainEvent> domainEvents);
  ...
```

Subscribe to domain events using a DomainEventDispatcher:

```java
public class DomainEventDispatcher {
    public DomainEventDispatcher(String eventDispatcherId,
                DomainEventHandlers eventHandlers,
                ...) {
...
}
```

Handle the events using DomainEventHandlers:

```java
public class RestaurantOrderEventConsumer {

  public DomainEventHandlers domainEventHandlers() {
    return DomainEventHandlersBuilder
            .forAggregateType("net.chrisrichardson.ftgo.restaurantservice.Restaurant")
            .onEvent(RestaurantMenuRevised.class, this::reviseMenu)
            .build();
  }

  public void reviseMenu(DomainEventEnvelope<RestaurantMenuRevised> de) {
```

### Transactional Commands

Transaction commands are implemented using transactional messaging.

Send a command using a CommandProducer:

```java
public interface CommandProducer {
  String send(String channel, Command command, String replyTo, Map<String, String> headers);
  ...
}
```

Subscribe to commands using a CommandDispatcher:

```java
public class CommandDispatcher {

  public CommandDispatcher(String commandDispatcherId,
           CommandHandlers commandHandlers) {
  ...
}
```

Handle commands and send a reply using CommandHandlers:

```java
public class OrderCommandHandlers {


  public CommandHandlers commandHandlers() {
    return CommandHandlersBuilder
          .fromChannel("orderService")
          .onMessage(ApproveOrderCommand.class, this::approveOrder)
          ...
          .build();
  }

  public Message approveOrder(CommandMessage<ApproveOrderCommand> cm) {
    ApproveOrderCommand command = cm.getCommand();
    ...
  }
```

### Kafka, Zookeeper and Mysql

light-tram-4j has the same dependencies as light-eventuate-4j on Kafka, Zookeeper and
Mysql. This must be started first before starting the CDC server below. The easiest way
to start all of the services is a docker-compose which is included in [light-docker][] 
repo.

For step by step guide, please refer to [getting-started][] tutorial of light-eventuate-4j

### Running CDC Server

In addition to a database and message broker, you will need to run the Tram CDC server. 
It reads events inserted into the database and publishes them to Apache Kafka. It is 
written using light-rest-4j. The easiest way to run this service during development is 
to use Docker Compose file in [light-docker][]. 

Once the above docker-compose is up and running, we can start CDC server.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-cdcserver-for-tram.yml up
```



[eventuate-tram-core]: https://github.com/eventuate-tram/eventuate-tram-core
[Todo list]: /tutorial/tram/todo-list/
[light-docker]: https://github.com/networknt/light-docker
[getting-started]: /tutorial/eventuate/getting-started/
