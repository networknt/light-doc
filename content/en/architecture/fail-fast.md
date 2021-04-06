---
title: "Fail Fast vs Fail Slow"
date: 2017-11-15T21:16:13-05:00
description: ""
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

### Why microservices need to fail-fast instead of fail-slow


Reserve resources

When an error occurs, whether it is an exception or a validation error or not, the service should stop processing and return that error status to the consumer. This will reserve processing power for the server.

Security concerns

Another reason we don’t output too much info in the response is security concerns. If a hacker tries to explore your service, he/she will construct invalid requests and see what the response error is. If there is too much information in the response, it can dramatically simplify the process. For normal consumer developers, they have centralized logging to help them to identify what is really happening when a simple error code is thrown.

Cascade failure

The more time for the service to process additional steps after an exception, the slower the response will be. Normally, the processing on the exception branches will be significantly slower than the happy path. This will cause the consumer to be waiting and the consumer of the consumer to wait. In a Java EE technology stack, it will cause the thread pool of the consumer to run out easily and the server will receive no response for new requests. The same failure will quickly propagate to the consumer of the consumer quickly and eventually bring down the entire system. That is why it is highly recommended to implement a circuit breaker and bull head.

### Why designers keep using fail-slow

There are several wrong assumptions in a designer's mind when they decide to use fail-slow.

#### The service will be called by UI

Most services in microservices architecture would be consumed by other services or web server /service aggregators. For these types of consumers, they would want fast-fail and that the response be as simple as possible. 


#### User need to have all the errors in one shot

Most arguments regarding the fail-slow design involve trying to give users all the validation errors so that users can fix all of them and resubmit. This actually comes from the old world of server side rendering which is based on JSP or Servlet. The response time is miserable on these systems and designers have to save every round trip to the server in order to improve user experience and reduce the load to the server as Java EE is blocking and throughput is constrained by the number of threads. With this mentality, the response is getting bigger and bigger and the response time is getting slower and slower.

When an organization is adopting microservices architecture, chances are they will build their UI with a mobile native app or single page app (Angular/React) to talk to the services built. In this type of design, the communication between client and server is based on a small JSON object or even an ProtoBuf binary and the response time is usually within 10 milliseconds. The entire validation design is changed from validate when submitting to validate when typing. Take a look at Google.com when searching: every character you type, there is a request and response between your browser and the google.com server. This goes to extremes as Google has the power and resources to do that. However, for most microservices based solutions, validating when a user moves the cursor from one field to another field on a form is a piece of cake. Here, each field is validated individually; the error message is small and with only one error at a time.


### Conclusion

Given the drawbacks of the fail-slow and the suitable use cases are replaced by the SPA. There
is no need to design your server to be fail-slow. For microservices architecture, one of the
principles is fail-fast. 

33


