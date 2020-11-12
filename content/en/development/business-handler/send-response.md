---
title: "Send Response"
date: 2020-08-30T23:28:29-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For rest APIs, we normally return a JSON string as the response body based on the business logic. 

### Status

In the status.yml config file in Status module of light-4j, we have defined most success response codes. Here is one of the examples. 

```
SUC10200:
  statusCode: 200
  code: SUC10200
  message: OK
  severity: NA
  description: The request is handled successfully
```

Here is an example to return 200 response if there is no additional information that needs to be returned for post and put requests. 

```
	// create a static String in the handler class. 	
    private static final String SUC10200 = "SUC10200";
```

```
        setExchangeStatus(exchange, SUC10200);
```

If there is an error, you need to define your error code in the app-status.yml file in the same format as the status.yml file.

For example. 

```
ERR10016:
  statusCode: 400
  code: ERR10016
  message: MISSING_HANDlER
  description: Could not find the handler for the endpoint %s.
```

As you can see that the desciption has a parameter, so we need to provide the parameter in the setExchangeStatus method.

```
        setExchangeStatus(exchange, ERR10016, "/v1/pets");
```

### Response

For any JSON response, we need to set the header for the content type and then use the sender to send the response in string. 

```
       exchange.getResponseHeaders().add(Headers.CONTENT_TYPE, "application/json");
       exchange.setStatusCode(200);
       exchange.getResponseSender().send("[\"Message 1\",\"Message 2\"]");
```

As you can see, you need to add the content type header and status code. 

