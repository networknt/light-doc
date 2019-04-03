---
title: "Security"
date: 2017-11-18T16:43:10-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When using message driven approach for service to service communication on top of light-eventuate-4j
framework, there are several security concerns that must be covered. For most customers, these issues
might be minor but for banks, health care, military and government they are major concerns that must
be addressed.   

The following discussion is based on Kafka as it is our first choice of messaging broker; however, 
the concept applies to other software package. 

There are two topics regarding to messaging security. The first is to ensure that only the right
producer and consumer can connect to the messaging broker. This is done by [ACLs][] and Kafka provides
a [command line interface][] for it. The second is data confidentiality and integrity.  

### User Authentication and Authorization

There are a lot of discussions on ACLs in Kafka and it is documented in [Security 101][]. 


### Data Confidentiality and Integrity

The area is new and it is very hard to find in-depth discussion. As you know we have light-oauth2 
which is an OAuth 2.0 provider but with a lot of enhancements to support microservices. One of the 
feature is public key certificate distribution API to allow different services to go to central 
location to get public certificate with a kid. It is the foundation for data confidentiality and 
integrity.

To protect data flow in Kafka or any message broker, we need to ensure that the data can be encrypted 
if necessary and we need to ensure that data can be signed so that it is not modified along the way.

Both can be done with PKI as you have mentioned but encryption is a little bit different due to 
performance issue. We are planning to implement something like TLS protocol to exchange the public 
key cert in a handshake and use an symmetric key to encrypt data. TLS is used to security the channel 
but for us we are using it to secure the payload only. This is very useful in our [light-hybrid-4j][] 
framework as we are not using anything in http but only the body stream for communication.

[ACLs]: https://developer.ibm.com/opentech/2017/05/31/kafka-acls-in-practice/
[command line interface]: https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Authorization+Command+Line+Interface
[Security 101]: https://www.confluent.io/blog/apache-kafka-security-authorization-authentication-encryption/
[light-hybrid-4j]: /style/light-hybrid-4j/



