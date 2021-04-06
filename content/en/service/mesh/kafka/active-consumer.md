---
title: "Active Consumer"
date: 2021-03-26T10:58:53-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When using the active consumer, the backend API/APP has full control over the consumption. The most simple process consists of three steps below. There are more endpoints for the position control and assignment and might be used by the backend API/APP. 

### Consumer Group

The first step is to create a consumer group with the following command. 

```
curl -k --location --request POST 'https://localhost:8443/consumers/testgroup' \
--header 'Content-Type: application/json' \
--data-raw '{"format": "jsonschema"}'
```

Result:

```
{"instance_id":"rest-consumer-id-11179f13-917f-44d6-91d9-da70a57b379e","base_uri":"/consumers/testgroup/instances/rest-consumer-id-11179f13-917f-44d6-91d9-da70a57b379e"}
```


### Subscribe

The second step is to subscribe to one or more topics with the previous command's returned instance. 

```
curl -k --location --request POST 'https://localhost:8443/consumers/testgroup/instances/rest-consumer-id-b92658a0-bc9c-4907-91e0-7c0d70e9fe68/subscriptions' \
--header 'Content-Type: application/json' \
--data-raw '{
  "topics": [
    "test1"
   ]
}'
```

There is no response body for this request. 

### Records

The third step is to retrieve the records. 

```
curl -k https://localhost:8443/consumers/testgroup/instances/rest-consumer-id-b92658a0-bc9c-4907-91e0-7c0d70e9fe68/records?format=jsonschema
```

Result:

```
[{"topic":"test1","key":"alice","value":{"count":2},"partition":0,"offset":495738},{"topic":"test1","key":"john","value":{"count":1},"partition":0,"offset":495739},{"topic":"test1","key":"alex","value":{"count":2},"partition":0,"offset":495740}]
```

