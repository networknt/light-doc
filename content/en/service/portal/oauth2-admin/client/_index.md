---
title: "Client Management"
date: 2020-07-13T21:36:57-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

To access any service from a client, you need to have a valid JWT token that contains a scope that matches one of the scopes defined in the service specification for the endpoint. Typically, the original client should be a single page application or a mobile application. However, in a microservices deployment, there will be service to service invocation, and the caller will be the client of the service provider. 

The original client will be authenticated and authorized by the code service through the authorization code flow. The JWT token will contain the user_id, roles, and possible custom claims for fine-grained authorization on each service in the invocation chain. Each intermediate service will obtain a secondary scope token with client credentials flow to indicate the immediate caller, and it is done automatically by the Http2Client in light-4j. 

For more details about the security architecture, please refer to [here](/architecture/security/)

For each client except the last service in the chain, it needs to be registered on the OAuth 2.0 provider as a client to get the token with scopes to access the next service in the chain. 

For all generated projects with light-codegen, the config folder contains certificates to verify the JWT token issued by the https://devoauth.lightapi.net behind the light-portal. 

To use the light-portal, you need to register a client and get a client_id and client_secret pair. Once you login to the https://dev.lightapi.net, you should see the OAuth menu on the dashboard. Click the client menu, and all the registered clients will be shown up in a table for the host you belong to. By default, the host is lightapi.net, and you can request a new host based on your email domain from your profile menu. 

### Register Client

To register a new client, go to the end of the table and you can see a plus button. Click the button, a client form will be shown up. Fill in the form and click the submit button, a regsitered client info will be returned. Copy the client_id and client_secret and keep it in a safe place. This is the only time, you can see the client_secret and there is no way to recover it. If you forget client_secret, you have to delete the client and re-register it again. 

To register a new client, go to the end of the table, and you can see a plus button. Click the button, and a client form will be shown up. Fill in the form and click the submit button, registered client info will be returned. Copy the client_id and client_secret and keep it in a safe place. This is the only time you can see the client_secret, and there is no way to recover it. If you forget client_secret, you have to delete the client and register another one.


### Update Client

All the fields for the client can be updated except client_id and client_secret. For each client in the table, you can click the update icon at the end of the row to show the update form. Once the update is done, click the submit button to save the client. 

### Delete Client

To delete a client, go to the end of the table row and click the delete icon. 

### Create Token

In most of the cases, JWT tokens will be retrieved in real-time during the service invocation. However, for dev testing, we need to get a test token to invoke the API manually or from testing tools. 

You can set the expired time to short or long, depending on your test. You can also set the scope to access any service that requires the scope. For authorization code token, you can set the user_id and roles to trigger the fine-grained authorization. Sometimes, you might need to set the custom claims for special authorization required in the service. 

