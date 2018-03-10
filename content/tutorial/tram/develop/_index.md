---
title: "Development API with light-tram-4j"
date: 2018-01-18T11:51:20-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


### Message/Event store:



Normal solution for the message/event store is use message broker for the message queue.The traditional solution is to use a distributed transaction that spans the database and the message broker.
However, distributed transactions are not a good choice for modern application.


Use a database table as a message queue


A straightforward way to reliably publish messages is to use a database table as a temporary message queue.
Each service that sends messages has an OUTBOX database table. As part of the database transaction that creates,
updates and deletes business objects, the service sends messages by inserting them into the OUTBOX table.
Atomicity is guaranteed since this is a local ACID transaction. Figure diagram shows how this works.



![Message-Queue](images/message-queue.png)



light-tram-4j define a message table which similar as normal message broker message by including header and payload:


```

CREATE TABLE message (
  ID VARCHAR(120) PRIMARY KEY,
  DESTINATION VARCHAR(1000) NOT NULL,
  HEADERS VARCHAR(1000) NOT NULL,
  PAYLOAD VARCHAR(1000) NOT NULL,
  PUBLISHED INT DEFAULT 0
);

```

ID: unique id for the message, primary key for the message table;

DESTINATION: destination topic for the message; it can be used for the message channel mapping

HEADERS: define the message header; Normally, header field will save key-value pair message header. for example

```
{"command_saga_id":"0000016106749406-0242ac1200050000","command_type":"com.networknt.example.sagas.ordersandcustomers.customer.command.ReserveCreditCommand","command_reply_to":"com.networknt.example.sagas.ordersandcustomers.order.saga.createorder.CreateOrderSaga-reply","command_saga_type":"com.networknt.example.sagas.ordersandcustomers.order.saga.createorder.CreateOrderSaga","command__destination":"customerService","ID":"000001610674941b-0242ac1200050000","command_saga_request_id":"0000016106749419-0242ac1200050000"}
```

PAYLOAD: define the message payload; payload field save json format message body. for example:

```
{"orderId":1,"orderTotal":{"amount":20},"customerId":1879729051024977}
```

PUBLISHED: this is fields is used for oracle pulling cdc. If user chose mysql or postgre as database, please ignore this field.

When user can light-tram-4j api to send message/event, this fields will use system default value 0; After oracle pulling cdc service published the message to kafka (or other message broker), the field value will be changed to 1 byn oracle pulling cdc service.


### Command message publish:


Use CommandProducer to publish command message:

```

 String send(String channel, Command command, String replyTo, Map<String, String> headers);

 String send(String channel, String resource, Command command, String replyTo, Map<String, String> headers);

```

Command object will be save as json format as message payload

channel, resource, replyTo will be added as header key-value pare;

headers key-value map will integrated with system default headers into the message header field.





### Event message publish:

Use DomainEventPublisher to publish event message:

```

  void publish(String aggregateType, Object aggregateId, List<DomainEvent> domainEvents);

  void publish(String aggregateType, Object aggregateId, Map<String, String> headers, List<DomainEvent> domainEvents);

```

domainEvents objects will be save as json format as message payload

headers key-value map will integrated with system default headers into the message header field.



System default hearders:


```
            .withHeader(Message.PARTITION_ID, aggregateIdAsString)
            .withHeader(EventMessageHeaders.AGGREGATE_ID, aggregateIdAsString)
            .withHeader(EventMessageHeaders.AGGREGATE_TYPE, aggregateType)
            .withHeader(EventMessageHeaders.EVENT_TYPE, event.getClass().getName())
```