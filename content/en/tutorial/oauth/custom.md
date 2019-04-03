---
title: "Custom Grant Types"
date: 2017-12-05T20:22:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-OAuth2 supports custom grant types and it is very easy to implement with a
special client type called trusted. The trusted client type is an add-on based on 
the standard public or confidential client types provided in OAuth 2.0 specification.

The key condition that to setup trusted client is that client and OAuth 2.0 provider
are deployed in the same organization so that OAuth 2.0 provider can trust the client
on certain custom grant types.

Currently, we have one custom grant type called client_authenticated_user.

### client_authenticated_user

Before you use this grant type, you have to create a new client through client API and
make sure the the client type is "trusted".

Once the client is registered, you should have a client_id and client_secret returned
from the endpoint. Please write down both especially client_secret as there is no way
to recover it. The client secret is generated and the hashed and salted result is saved
into the database table. So there is no way we can retrieve it later.

For information on client registration, please refer to [client tutorial][]

To start the light-oauth2 services in a docker-compose, please refer to [How to start services][]

Once the services are up and running, let's create a brand new client and set the client
type as trusted. 

Here is the curl command line. 

```
curl -k -H "Content-Type: application/json" -X POST -d '{"clientType":"trusted","clientProfile":"mobile","clientName":"Trusted Client Demo","clientDesc":"A demo client that is trused","scope":"t.r t.w","redirectUri": "https://localhost:8080/authorization","ownerId":"admin"}' https://localhost:6884/oauth2/client
```

The following is the example response and the clientId and clientSecret in your command
would be different. Please write down the clientId and ClientSecret as subsequent commands
need them.
 

```
{"clientId":"96058f8e-6c03-4b2f-b30e-25b8d093d6d2","clientSecret":"mkFLT99FS260txCk9zKt5A","clientType":"trusted","clientProfile":"mobile","clientName":"Trusted Client Demo","clientDesc":"A demo client that is trused","ownerId":"admin","scope":"t.r t.w","redirectUri":"https://localhost:8080/authorization","createDt":"2017-12-06","updateDt":null}
``` 

Given that we have created a trusted client, we are going to use this client to access
token endpoint in order to generate a token with client_authenticated_user grant type.

Here is the command.

```
curl -k -H "Authorization: Basic 96058f8e-6c03-4b2f-b30e-25b8d093d6d2:mkFLT99FS260txCk9zKt5A" -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "grant_type=client_authenticated_user&userId=admin&userType=Employee&transit=12345" https://localhost:6882/oauth2/token

```

And the response is something like this.

```
{"access_token":"eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTUxMjUyODAzNSwianRpIjoiZE82c0hNZ3RqTENZUmVnT3QzUURyUSIsImlhdCI6MTUxMjUyNzQzNSwibmJmIjoxNTEyNTI3MzE1LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6ImFkbWluIiwidXNlcl90eXBlIjoiRW1wbG95ZWUiLCJjbGllbnRfaWQiOiI5NjA1OGY4ZS02YzAzLTRiMmYtYjMwZS0yNWI4ZDA5M2Q2ZDIiLCJzY29wZSI6WyJ0LnIiLCJ0LnciXSwidHJhbnNpdCI6IjEyMzQ1In0.bDL7LCAEWSZiJPJGfMyr7DJeO6DKnKS5dEp_IRGiWHOaYv5sck6dExE-JMI4pwroY6tVuG7-fpZ3QobkAUdwOfInqFbQcz1NdusiaBmTty8-60dZQMjsORzVv41cnMRWxdmKL42GH1vcbkCG1YP9orMUEbbQ0wPILfnkNpD00hcIWIWfNMKlGml6vGxFtbTy2hIoL-MUObwBelDSveQ5hjNIavtw7jj5Uq3XfimIzN0LeTAe8kzGc4R_PQK5LnaS2Qk8Ys_KnoY_G6SKRB1JuvBwSiIjHYYvmfSeJz3kVGYaivVQHMMpafRnuddkuaaxe1IHTzLtUdQuiLCUZJ4lxA","refresh_token":"d9bd57f7-cd83-4aed-946a-30213c0703de","token_type":"bearer","expires_in":600}
```

You can decode the token at jwt.io site. 

For more information on how to get the token with this custom grant type, you can check the
test cases at https://github.com/networknt/light-oauth2/blob/master/token/src/test/java/com/networknt/oauth/token/handler/Oauth2TokenPostHandlerTest.java#L1030

Also, there is an standalone example built for one of our customers at https://github.com/networknt/light-example-4j/blob/master/client/tomcat/src/main/java/com/networknt/client/Http2ClientExample.java 

[client tutorial]: /tutorial/oauth/client/
[How to start services]: /tutorial/oauth/start/
