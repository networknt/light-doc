---
title: "Integration Test"
date: 2018-01-06T20:30:27-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Integration tests are located in each sub-project src/integration folder and the filenames are ended with IT
instead of Test like Unit Tests. For more information on how to write integration tests in general, please 
refer to [Integration Test Tutorial][]. As light-eventuate-4j framework contains a lot of moving parts and 
depends on Kafka, Zookeeper, Mysql and CDC server to run, most test cases in the project are actually integration tests instead of unit tests. 

### Integration Test

*  Following the steps on [getting started][]

  To start event store, cdc service, and command/query side service(optional)

*  Send request from command side the publish events:

 From postmand, send post request:

   URL: http://localhost:8083/v1/todos;

   Headers:[{"key":"Content-Type","value":"application/json","description":""}];

   Body: {"title":" this is the test todo from postman1","completed":false,"order":0};


   Response:
{
  "done": true,
  "cancelled": false,
  "completedExceptionally": false,
  "numberOfDependents": 0
};

 This request will send requests which will call backe-end service to generate a "create todo" event and publish to event store.

 The event sourcing system will save the event into event store.

 CDC service will be triggered and will publish event to Kafka:


*  Subscribe to the event and process the event on the query side:

The event sourcing system will subscribe to the events from event store and process user defined event handlers.

For the todo-list example, the event handle will simply get the event and save the latest todo info into the local TODO table.


From Postman or from brower, send the GET request:

http://localhost:8082/v1/todos:

Reponse:
[
  {
    "0000015c8a1b67af-0242ac1200060000": {
      "title": " this is the test todo from postman1",
      "completed": false,
      "order": 0
    }
  }
];

[Integration Test Tutorial]: /tutorial/common/test/integration-test/
[getting started]: /tutorial/eventuate/getting-started/
