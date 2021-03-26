---
title: "SPNEGO/Kerberos"
date: 2018-05-24T09:20:17-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-oauth2 authorization code service supports both SPNEGO/Kerberos and Basic Authentications in HTTP get endpoint /oauth2/code. After being successfully authenticated, an authorization code will be directed back to the Single Page Application. If the SPA does not handle the redirected URI path, then the BFF(backend for frontend) will get the authorization code to complete the authentication and authorization flow on behalf of the SPA. In this flow, the user credentials won’t be revealed to the SPA and the access token won’t be revealed to the SPA running on the browser. If the SPA knows the token, it is subject to XSS attack which is very hard to prevent given the current Javascript and Single Page Application ecosystem.

### Authentication Flow

You can securely negotiate and authenticate HTTP requests for code services in the light-oauth2 by using the Simple and Protected GSS-API Negotiation Mechanism (SPNEGO) as the web authentication service.

SPNEGO is a standard specification that is defined in The Simple and Protected GSS-API Negotiation Mechanism (IETF RFC 2478) and SPNEGO-based Kerberos and NTLM HTTP Authentication in Microsoft Windows (IEFT RFC 4559).

When SPNEGO is enabled in light-oauth2 authorization code service, SPNEGO negotiation will take the priority. If SPNEGO is not enabled or the user agent (web browser) doesn't understand the negotiate header, then basic authentication will be used as a fallback authentication mechanism. In that case, a popup window will appear and username/password will be required. 

In addition to light-oauth2 authorization code service, some external components are required to enable the operation of SPNEGO. These external components include:

* Microsoft Windows Servers with Active Directory domain and associated Kerberos Key Distribution Center (KDC). Microsoft Active Directory acts as both Active Directory/LDAP server and Kerberos KDC server. A service account must be created in Active Directory for light-oauth2 server, say REALM\svc_oauth. An SPN for light-oauth2 code service which will bind the domain name of your server to this server account needs to be created. If your server is running on https://code.lightapi.net it should be HTTP/code.lightapi.net

* A Single Page Application running on a browser. Microsoft Internet Explorer, Google Chrome and Mozilla Firefox are browser examples. Any browser that is being used must be configured to use the SPNEGO web authentication mechanism. If windows user is logged into domain REALM and goes to https://code.lightapi.net, the browser will lookup an SPN based on name HTTP/code.lightapi.net, request a Kerberos ticket from Active Directory/KDC and send it to the server in an Authorization header as per SPNEGO specification

* The authentication of HTTP requests is triggered by the user (the client-side), which generates a SPNEGO token. The light-oauth2 code service receives this token and then decodes and retrieves the user identity from the SPNEGO token. The identity is then used to make authorization decisions. 

* As you can see LDAP is not involved in this authentication flow (although it can be used for authorization as a next step)

### Authorization Flow

SPNEGO web authentication is a server-side authentication solution in the light-oauth2 code service. Client-side applications are responsible for generating the SPNEGO token for use by SPNEGO web authentication. Once the user identity is verified and confirmed, the next step would be to retrieve user profile info and put into the JWT token for fine-grained authorization. For some of the organizations, group info from Active Directory/LDAP would be enough. However, some organizations have other systems that hold user profile info. The current implementation has the LDAP integration to retrieve user info only. Other customized authorization implementation will be provided and maintained by the customers. 


### References

SPNEGO/Kerberos is a huge topic and we cannot explain it easily in this article. The following list of reference documents will help you to understand the details. 

Specifications:

* https://tools.ietf.org/html/rfc4559
* https://tools.ietf.org/html/rfc4178

Authentication

* https://dzone.com/articles/do-not-publish-configuring-tomcat-single-sign-on-w
* http://java.sun.com/products/jndi/tutorial/ldap/security/gssapi.html

Authorization

* http://spnego.sourceforge.net/enable_authZ_ldap.html
* http://spnego.sourceforge.net/user_access_control.html
* http://spnego.sourceforge.net/LdapUserInfoExample.java
* http://spnego.sourceforge.net/spnego.policy
* http://spnego.sourceforge.net/ldap_query_names_groups.html
* http://www.kouti.com/tables/userattributes.htm


