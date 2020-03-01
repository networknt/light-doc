---
title: "Light Oauth2 Tutorial"
date: 2017-11-10T14:26:28-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This document will give you step-by-step guidelines on how to use light-oauth2. There are two types of services in light-oauth2. The first category contains code/authorize, token, key, and provider that will be accessed from clients during the runtime. The provider service is not directly accessed from the clients, but it connects federated light-oauth2 services to form a mesh of the OAuth 2.0 providers. Othe services like the user, client, and service are connected from the light-portal to make configuration changes to the light-oauth2 services. These services are protected by OAuth 2.0 authentication and authorization. By default, the security is partially disabled on these services out of the box so that users can easily test these services to learn how to use them. In later sections, there are steps to enable all security with config files.

* [How to generate long lived access token][]

* [How to start services][]

* [Authorization Code][]

* [Token Endpoint][]

* [Service Registration][]

* [Client Registration][]

* [User Management][]

* [Key Distribution][]

* [Client Authenticated User][]

* [Provider Service][]

* [Deploy to Kubernetes Cluster][]

* [Deploy to Openshift Cluster][]

* [Integrate Kerberos/SPNEGO on Mac][]

* [React Login View](/tutorial/oauth/login-view/)

* [Local Form Authentication](/tutorial/oauth/form-auth-local/)

* [Test Cloud Deployment and Access](/tutorial/oauth/test-cloud/)

[How to generate long lived access token]: /tutorial/oauth/longlive/
[How to start services]: /tutorial/oauth/start/
[Authorization Code]: /tutorial/oauth/code/
[Token Endpoint]: /tutorial/oauth/token/
[Service Registration]: /tutorial/oauth/service/
[Client Registration]: /tutorial/oauth/client/
[User Management]: /tutorial/oauth/user/
[Key Distribution]: /tutorial/oauth/key/
[Client Authenticated User]: /tutorial/oauth/custom/
[Provider Service]: /tutorial/oauth/provider/
[Deploy to Kubernetes Cluster]: /tutorial/oauth/kubernetes/
[Deploy to Openshift Cluster]: /tutorial/oauth/openshift/
[Integrate Kerberos/SPNEGO on Mac]: /tutorial/oauth/kerberos/
