---
title: "Coarse-grained Authorization and Fine-grained Authorization"
date: 2019-04-18T12:49:38-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In Light, we are using OAuth 2.0 to authorize the client request to the service with JWT token that contains two different claims. 

### Scopes

Scopes work at the technical level to protect individual endpoints within the service, and we call it coarse-grained authorization. For example, you have a service that provides four endpoints to select, insert, update and delete a resource. You can define four scopes and sign them to four endpoints. 

In the light-rest-4j framework, both Swagger 2.0 and OpenAPI 3.0 specifications support scope definition per endpoint, and we are loading the specification during the runtime to verify the scope of the endpoint against the scopes in the JWT token client retrieved from the OAuth 2.0 server. 

In the light-graphql-4j, there are only several endpoints and scopes usually only used to determine if the client can access query or mutation or subscription. 

In the light-hybrid-4j, we have scopes defined for each endpoint similar to the light-rest-4j. 

Since the scopes in the JWT token are used for the framework to determine if the client can access the certain endpoint, we need to make sure that client cannot set scopes on their own without any verification. 

In the light-oauth2 implementation, we have clients, services and clients to service endpoints relationship registered so that the light-oauth2 token service can calculate the scopes. It is the recommended approach for most customers, and it is the easiest way to handle the scopes in the JWT issuing. 

Sometimes, there might be too many scopes in the calculated result and the token generated might have a very broad access level to meet some organization's security policy. In this case, we provide two ways to reduce the scopes in the JWT token. 

* Pass in intention

It allows the client to tell the OAuth 2.0 token endpoint the service Id it wants to access so that only the scopes for the service will be included instead of all the scopes associated with services the client can access. This is the feature that light-oauth2 has implemented. 

* Pass in scopes

For most OAuth 2.0 implementation, this is the only option available as service registration is not mandatory in the OAuth 2.0 specification. When this approach is used, the scopes must be registered along with the client registration. And the OAuth 2.0 token service must verify the scopes passed in from the client is a subset of the registered scopes. 


**Note:**  You cannot blindly put the scopes passed from the client into the JWT token without verification. It basically allows the client to control which endpoints they can access. 


### Custom Claims

As you can see,  the scopes verification is done by the framework at the technical level. For most services, this is not enough. Specific fine-grained authorization might be necessary within the business context. Unlike the scope/endpoint level of authorization, this level of authorization must happen within the business context that means it might be different per service. To implement this level of authorization, it is recommended implementing with middleware handlers like other cross-cutting concerns in Light. In this case, the fine-grained authorization handler(s) will be wired in after the technical middleware handlers and before the business handler. 

Strictly speaking, the scopes are part of the custom claims; however, it is kind of standard in the OAuth 2.0 implementation although it is not defined in the OAuth 2.0 specification. 

When handling the custom claims for fine-grained authorization, the same approach can be taken like scopes. 

There are two types of custom claims: 

* client custom claims

These are attributes associated with the client. For example, the application Id or the transit the application belong to, etc. The fine-grained authorization handler can use the info to limit the number of columns or number of rows to return to the client based on the application id or transit. 

* user custom claims

These are attributes associated with the login user. For example, the user's role or the user's geolocation, etc. The fine-grained authorization handler can grant access based on the user's role or geolocation. 

For example, if you are a manager role, you can transfer the amount higher than 10K. If you are located in Asia, you cannot access the accounts opened in America. 

From an implementation point of view, these attributes would be somewhere in one or more database tables. The light-oauth2 has an authorizer interface and allow you to plugin the integration logic. If you are using another implementation of OAuth 2.0, you can find out how to integrate with your existing service or database from the documentation. 

With the integration between the OAuth 2.0 server and your own database or service, nothing needs to be done at the client level. Depending on the policies on the OAuth 2.0 server, the client custom claims and user custom claims will be automatically included in the generated JWT token. 

If only client custom claims are used, it is possible that the custom claims can be registered with the client like the scopes. 


Unlike scopes, there is a possible case that the scopes might be limited, custom claims shouldn't be sent from the client. This is a very dangerous practice if there is no verification implemented on the OAuth 2.0 server which is the case in most situations. If the OAuth 2.0 server can verify the custom claims, it can just simply put them into the token itself. 


If your OAuth 2.0 implementation doesn't support the integration to register the custom claims with the client and retrieve the user custom claims from user database, LDAP or user management system, then you can do the integration in the fine-grained authorization handler. 

**Note:** You cannot let the client pass the custom claims to the OAuth 2.0 server to create the JWT token. This gives the client the freedom to claim any attributes in the JWT token. 


### Summary

I wrote this article because I saw that some users requested to add custom claims into our client module to allow their OAuth 2.0 server to issue JWT tokens with custom claims for fine-grained authorization. This is the wrong approach that we must point out as this mistake will have serious consequences. 

The most common argument that I have heard is that the OAuth 2.0 server they are using doesn't support integration. In this case, it would be wise to skip the custom claims from the client/JWT and implement the standard fine-grained authorization middleware handler that is included in all the services. Within the middleware handler, you can integrate with any subsystems that support the client attributes and user attributes. In this way, it is still that the service decides the authorization, not the client. In the client credentials flow, the client_id is in the JWT token so that you can use it to look up the client attributes that support the fine-grained authorization. In the authorization code flow, both client_id and user_id will be in the JWT token to look up the client attributes, and user attributes to support the fine-grained authorization at the client level and user level. 

The other argument I've heard is that our clients and services are all in the same organization and there is auditing procedure in place to ensure that client code is sending the correct custom claims all the time. This simply cannot be enforced, and in a real microservices architecture, the number of services and clients are increasing very fast. Eventually, your service will be accessed by the party you have no control or even outside of your organization. As the client is passing in the custom claims, it is very easy for hackers to explore the security hole. 

The concept is very hard to understand for developers as most of them don't have security background. Let's use a real world physical security analogy to explain it. 

You have a building with a lot of rooms, and each room has a security door that needs a pass to access. On the first floor, there is a security desk that issue passes with permissions to access each room. An employee want to access the secure rooms, come to the security desk and ask for a pass to access room 10 with his employee card as an ID. Since the security desk doesn't have a phone available, the security guy cannot verify with the room owner if the employee can access the room or not. He/she just issue a pass for room 10 as the employee claims he has access. In this way, every employee can access any room if they want to. 

To enforce the proper access rules given the security desk cannot do so, it might be better not to issue the pass from the security desk to the employee but allow the room owner to verify the employee permission at the door. 

Light is a security first design, and we have implemented a lot of features to make the traditional OAuth 2.0 specification work with microservices architecture. Our principle is centralized policy management and distributed policy enforcement. In our architecture, the policies are managed in the light-oauth2 services most to the cases. If your OAuth 2.0 server doesn't support customization, please pass this responsibility to the service implementation with middleware handlers to share between multiple services. You cannot pass the security responsibility to the clients. If all clients decide the scopes and custom claims, then there is no security at all even you have the scopes and custom claims verification implementation on the service side, as clients can simply send anything they want. 

