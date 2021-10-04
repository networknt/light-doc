---
title: "SPA Component"
date: 2021-10-03T15:38:33-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Although any light-4j service can host a SPA application, it is only for demo and development purposes. For a real SPA application that accesses backend APIs, some cross-cutting concerns and security need to be addressed.

By default, all light-4j services have JwtVerifierHandler wired in to verify the JWT token and the best grant flow for SPA is the authorization code flow. The following is the diagram with all the related components for the light portal, which is the main UI for the light platform.


![spa-component](/images/spa-component.png)


1. Mobile applications are running on mobile devices and need to access backend API for complete functions. Single Page Applications are running within web browsers and using javascript to access remote services for most of the functionalities. When SPA or Mobile application is loaded, it will try to access the backend API with light-router. 

Here the light-router is a micro gateway that is responsible for the following: 
* Serve the SPA application so that there is no need to config CORS as the site and the APIs are on the same site.
* Handle the JWT token retrieval based on the authorization code as a server is needed to keep the client_id and client_secret. 
* Aggregate all backend APIs to a single access point for simplicity and security.
* Address all other cross-cutting concerns like rate limiting, IP whitelisting etc., for Internet-facing SPA or Mobile.
* Prevent CSRF attack with CSRF token inside the JWT token

2. If there is no token in the secure cookies within the request to the light router, the request will be redirected to the OAuth Code service for users to authenticate with credentials. When the OAuth Code service is integrated with the LDAP with SPNEGO, there is no need to provide any credentials from users. The network Id used to log in to your desktop or laptop will be used to authenticate the user behind the scene with SSO.


3. After the authentication from the OAuth Code service with the user's credentials, an authorization code will be returned to the SPA or Mobile app, and the same request will be sent to the light router with the authorization code. 

4. The light router will use the authorization code along with the client_id and client_secret in the config to get a JWT token. This token will have the client_id, user_id and role so that subsequent invocation to the API will receive information for audit and fine-grained authorization.

5. The JWT token will return to the light router from the OAuth Token service along with a refresh token and token expiration info. 

6. The light router sends back the JWT token and the refresh token to the SPA or Mobile app. For a SPA app, they will be sent with secure cookies so that the Javascript cannot access them. When the token expires, the light-router will use the refresh token to renew a new JWT access token and send it to the browser. 

7. All subsequent API calls to the portal query APIs will be sent to the light router from the SPA.

8. When a read-only Control Plane or Config Server request is sent, the light router will discover the Portal Query service and forward the request to it. 

9. The Portal Query service will invoke the Control Plane service to access the read-only APIs. 

10. The Portal Query service will invoke the Config Server service to access the read-only APIs.

11. When an update Control Plane or Config Server request is sent, the light router will discover the Portal Command service and forward the request to it.

12. The Portal Command service will invoke the Control Plane service to access the write APIs.

13. The Portal Command service will invoke the Config Server service to access the write APIs.

14. The Control Plane will use the light scheduler to execute the health check tasks.




