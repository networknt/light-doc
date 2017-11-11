---
title: "Customize Status"
date: 2017-11-10T18:50:54-05:00
description: "How to customize the output format for Error Status response"
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-4j Status module is an object that model the error response and constructed from
error code defined in status.yml config file. The framework provide some error code and
we are expecting the API itself provide some service specific error code. For more detail
on [Status][] module, please check the cross-cutting concerns section. 

The default output of error status is a JSON string with several properties flattened in
a map. Here is an example.

```json

{
  "statusCode": 401,
  "code": "ERR10001",
  "message": "AUTH_TOKEN_EXPIRED",
  "description": "Jwt token in authorization header expired"
}
```

One of our customers has a special format for error code along with some other meta data
and they ask if the output format can be changed. And the following is the solution for
users want to use customized serializer. 

First we provide an interface called StatusSerializer

```java
package com.networknt.status;

/**
 * Interface to allow custom serialization for a Status.
 * 
 * Framework users can define their own format to return an error message to a consumer
 * 
 * @author Dan Dobrin
 */
public interface StatusSerializer {
	/**
	 * Serialize the status and provide a custom format in the iomplementing class
	 * 
	 * @param status The status to be serialized
	 * @return the format Status object, to be serialized and returned to the consumer
	 */
	public String serializeStatus(Status status);
}

```

Second you need to implement above interface with your format. Here is an example.

```java
public class ErrorRootStatusSerializer implements StatusSerializer {

	@Override
	public String serializeStatus(Status status) {
		return "{ \"error\" : {\"statusCode\":" + status.getStatusCode()
        + ",\"code\":\"" + status.getCode()
        + "\",\"message\":\""
        + status.getMessage() + "\",\"description\":\""
        + status.getDescription() + "\"} }";
	}

}

```

Third you add the mapping between the interface to implementation in service.yml

```yaml
# Singleton service factory configuration
singletons:
  - com.networknt.status.StatusSerializer:
    - com.networknt.status.ErrorRootStatusSerializer
```

Now, the ErrorRootStatusSerializer will override the default one and the output will
be something like this.

```json
{
  "error": {
    "statusCode": 401,
    "code": "ERR10001",
    "message": "AUTH_TOKEN_EXPIRED",
    "description": "Jwt token in authorization header expired"
  }
}
```

As you can see that there is an error object that contains all the properties in Status
object. You can change the format to anything you want and the IoC service module will
be responsible to load your implementation from service.yml config file. 

[Status]: /concern/status/
