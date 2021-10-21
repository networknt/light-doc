---
title: "Get Request Info"
date: 2020-08-30T23:28:43-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Most developers who are working on Light are building the final business handlers. Their job is to get the HTTP query parameters, path parameters and request body in the generated request handlers and create the corresponding response based on the input above. When users choose different HTTP frameworks, they need to know how to get the details from the request and then send the response. This document will be focusing on the request details. 

### Header

To read the header, you can use the following method. 

```
String contentType = exchange.getRequestHeaders().getFirst(Headers.CONTENT_TYPE);
```

The Headers class defines a lot of HttpString objects for standard headers. If you cannot find yours, you can create a constant to define yours. It is faster than creating HttpString every time it is used. 

```
public static final HttpString BASIC = new HttpString("Basic");
```

Light-4j has a class [HttpStringConstants][] that defines several HttpString it uses. 


### Query and Path Parameters

Light-4j is built on top of the Undertow Core HTTP server, and it has different methods to get query parameters and path parameters. 

To get all the query parameters.

```
        Map<String, String> params = new HashMap<>();
        Map<String, Deque<String>> pnames = exchange.getQueryParameters();
        for (Map.Entry<String, Deque<String>> entry : pnames.entrySet()) {
            String pname = entry.getKey();
            Iterator<String> pvalues = entry.getValue().iterator();
            if(pvalues.hasNext()) {
                params.put(pname, pvalues.next());
            }
        }
        if(logger.isDebugEnabled()) logger.debug("params", params);
        String clientId = params.get("client_id");

```

To get an individual query parameter or path parameter.

```
       String clientId = exchange.getQueryParameters().get("clientId").getFirst();
```

The above assumes that validator handler is enabled to guarantee that clientId exists. Otherwise, you might get NullPointerException. 

The above clientId is a path parameter and you can use the same getQueryParameters to get all of them. We have merged all parameters together in the handler when matching the template. 

The url for the above path paramter. 

```
ClientRequest request = new ClientRequest().setMethod(Methods.DELETE).setPath("/oauth2/client/59f347a0-c92d-11e6-9d9d-cec0c932ce03");
```

### Cookies

To get cookies.

```
            Map<String, Cookie> cookies = exchange.getRequestCookies();
            if(cookies != null) {
                Cookie cookie = cookies.get(ACCESS_TOKEN);
                if(cookie != null) {
                    jwt = cookie.getValue();

```

To set cookes in a response.

```
List scopes = setCookies(exchange, result.getResult(), csrf);
```

For more detail, please take a look at the light-spa-4j [StatelessAuthHandler][].

### Request Body

For most users, the request media is application/json or application/x-www-form-urlencoded, and we have parsed body to a map and attach it to the exchange with the [BodyHandler][]. 


#### application/json

```
Map<String, Object> body = (Map<String, Object>)exchange.getAttachment(BodyHandler.REQUEST_BODY);

```

For some users, they have a POJO to model the request. The following method can be used to convert the map to a Java Object. 

```
Client client = Config.getInstance().getMapper().convertValue(body, Client.class);
```

Sometimes, the request body is not a map but a list, then we need to use the following method to get the list. This is purely based on the specification for the request. 

```
List<String> endpoints = (List)exchange.getAttachment(BodyHandler.REQUEST_BODY);
```

Or

```
List<Map<String, Object>> body = (List)exchange.getAttachment(BodyHandler.REQUEST_BODY);
```

#### application/x-www-form-urlencoded

This is the normal web form submission, and the result will be converted to a map or list if the body handler is configured in the chain just like the JSON above. 

In case you don't have BodyHandler in the chain for the endpoint, you can use the following code to parse the form data manually. 

```
        // get the form from the exchange
        final FormData data = exchange.getAttachment(FormDataParser.FORM_DATA);

        final FormData.FormValue jClientId = data.getFirst("client_id");
        final FormData.FormValue jRedirectUri = data.getFirst("redirect_uri");
        final FormData.FormValue jState = data.getFirst("state");
        final FormData.FormValue jRemember = data.getFirst("remember");
        final String clientId = jClientId.getValue();
        final String remember = jRemember == null ? null : jRemember.getValue();  // should be 'Y' or 'N' if not null.
        String redirectUri = jRedirectUri == null ? null : jRedirectUri.getValue();
        final String state = jState == null ? null : jState.getValue();
        if(logger.isDebugEnabled()) {
            logger.debug("client_id = " + clientId + " state = " + state + " redirectUri = " + redirectUri + " remember = " + remember);
        }

```

For most users, we recommend to add the BodyHandler to the endpoint in the handler.yml and get the map from the exchange attachment. 

#### multipart/form-data

For users who want to send binary data from a client to a server, you can use this media type. The BodyHander will convert this to a stream and put it into the exchange as an attachment. Here is the code in the BodyHandler.

```
InputStream inputStream = exchange.getInputStream();
exchange.putAttachment(REQUEST_BODY, inputStream);
```

If user wants to upload large binary file from API to API call, below is the sample code:

```text
            ClientRequest request = new ClientRequest().setPath(requestUri).setMethod(Methods.POST);
            
            request.getRequestHeaders().put(Headers.CONTENT_TYPE, FORM_DATA_TYPE);
            request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
            InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream("sample.pdf");
            HashMap<String, Object> requestMap = new HashMap<>();
            requestMap.put("name", "test_sample.pdf");
            requestMap.put("profileFile", IOUtils.toByteArray(resourceAsStream));
             //customized header parameters 
            connection.sendRequest(request, client.createClientCallback(reference, latch, ByteBuffer.wrap(SerializationUtils.serialize(requestMap))));
```

Or user can try test from postman by using  form-data:

In postman, set method type to POST.

Then select Body -> form-data -> Enter your parameter name (file according to your code)

And on right side next to value column, there will be dropdown "text, file", select File. choose your image file and post it.


![form-data](/images/postman.png)

For details, please take a look at the [pdf example][] in the light-example-4j. 

[pdf example]: https://github.com/networknt/light-example-4j/tree/master/client/pdf
[BodyHandler]: https://github.com/networknt/light-4j/blob/master/body/src/main/java/com/networknt/body/BodyHandler.java
[HttpStringConstants]: https://github.com/networknt/light-4j/blob/master/http-string/src/main/java/com/networknt/httpstring/HttpStringConstants.java
[StatelessAuthHandler]: https://github.com/networknt/light-spa-4j/blob/master/stateless-auth/src/main/java/com/networknt/auth/StatelessAuthHandler.java

