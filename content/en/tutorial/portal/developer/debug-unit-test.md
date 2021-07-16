---
title: "Debug with Unit Test"
date: 2020-03-21T10:35:10-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When developing light-portal services, it is crucial to ensure that each endpoint works before loading them from the hybrid server. The best approach is to debug and test with JUnit tests within each service. In this tutorial, we are going to walk through the process with light-portal user-command and user-query services. 

This approach has its limitations. It requires that the endpoint tested is working independently without access to other services. If it requires additional services, you need to start one of the hybrid-command or hybrid-query servers. For example, when you are running updateUserById on the user-command service through JUnit, you might need to start the hybrid-query server with user-query service loaded for the query. Another example is when debugging the streams processing on the user-query, you might need to either run a JUnit test case from user-command to publish an event to Kafka or start a hybrid-command server and push an event from curl command line or postman. 


### Environment


##### Confluent Platform

In different combinations, we need to ensure the local environment is ready for debugging or testing. 

Almost all services in light-portal need Kafka for event sourcing and CQRS. We need to start the Confluent platform locally. I have it installed in my tool folder. 

```
cd ~/tool/confluent-6.1.0/bin
confluent local services start
```

To check the status.


```
confluent local services status
```

To stop the Confluent platform.


```
confluent local services stop
```

##### Consul

When you need two or more servers to interact with each other, it is better to have a Consul docker instance running locally for flexibility. Of course, you can use the DirectRegistry in the service.yml to pin each server to a specific IP and port number. 

To start a Consul instance with Docker.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

##### Light-oauth2

If there are any interactions between the command side and the query side, we need the light-oauth2 services running locally to provide client credentials tokens. For all portal services, we have security enabled by default, as most of them need the user_id and roles in the JWT token to complete the logic. For command side services, we need the user_id in JWT to generate the eventId object to indicate the owner of the events.

To start the light-oauth2 and register them to the Consul. 

```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose up -d
```

### Independent Endpoint

With most of the user services depending on the user_id and roles from the JWT token, please set hybrid-security.yml enableVerifyJwt to true. It requires all tests must contain authorization header with a long-lived token. 

Here is a valid token for testing. 

```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTkwMDE3MDQ3OSwianRpIjoiZ2R2OUNsOGJyR0dUQWhVbjVsZnhfUSIsImlhdCI6MTU4NDgxMDQ3OSwibmJmIjoxNTg0ODEwMzU5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHVAZ21haWwuY29tIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJyb2xlcyI6InVzZXIgbGlnaHRhcGkubmV0IGFkbWluIiwic2NvcGUiOlsicG9ydGFsLnIiLCJwb3J0YWwudyJdfQ.rZOj27je8e6HEb1XvIm34zDjVUUaBUHXqvTyQWnw2va1Xbe2_6nX0hk7VqNpdIWUL5-4tcdyQ4iVVGtEpZWIhY9s2vtoL2Zsc1e5WxViqZ8cJ00zyTU1aeOuXjVWyOc2HC754xLl4WS0se6TEpi8Xs5gVrczaNvuvnrVns1KC2q_0cN574EGmT8Fbi3EP3j_kMVXf_m7NdxaV2PREbY_bejlgqsH6bWE0CSIgt4olAlgIZUHifZ7mFElXdek7MZ2ywaAitmoyMWYgUhPzoX_EYcCgrusU2KrOFfm-9CGwZ1Wf2LGawfbcnBDRjW28pzzBaHINADXrYOV8cy7NWGEZg
```

From the jwt.io site, you can see the claims of the token. 

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1900170479,
  "jti": "gdv9Cl8brGGTAhUn5lfx_Q",
  "iat": 1584810479,
  "nbf": 1584810359,
  "version": "1.0",
  "user_id": "stevehu@gmail.com",
  "user_type": "EMPLOYEE",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "roles": "user lightapi.net admin",
  "scope": [
    "portal.r",
    "portal.w"
  ]
}
```

### Debug

Most query side endpoints should work independently. We are going to use the user-query service for this demo. The detail steps to set up the Run/Debug configuration is documented [here](https://doc.networknt.com/tutorial/common/debug/idea/)

The following is a video walkthrough. 


{{< youtube Z0tm3Pwk60g >}}

