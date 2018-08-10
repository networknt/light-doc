---
title: "Kerberos/SPNEGO"
date: 2018-08-07T14:08:08-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-oauth2 supports Kerberos/SPNEGO with the built-in default authentication/authorization authenticator. This allows the user only sign-on the desktop once and call OAuth2 authorization code flow with Single Sign On on the browser.

For the light-oauth2 server and AD/KDC configuration, please refer to [default auth][]. Once you have light-oauth2 and AD/KDC servers ready, you can perform an integration test on you local Mac Book Pro with SSO to confirm that all configurations are working. 

### Prerequisite

Here is the steps to do the integration test. 

1. Assume that AD/KDC are ready

You can set up a single testing server on the domain controller in Microsoft Windows Server. Please refer to [default auth][] for more details.

2. Start the light-oauth2 service with Docker

The configuration for light-oauth2 is a little complicated when integrating with Kerberos/SPNEGO. You can find a config folder which works for release 1.5.18 in https://github.com/networknt/light-config-test/tree/master/light-oauth2/dna-spnego

Please note that the configuration files in the above folder contain files designed for example.com site, you need to make corresponding updates for your deployment. 


### Start OAuth 2.0 services

Let's assume that we have another folder of light-oauth2 config files with the proper config to point to the AD/KDC server within the organizion. 

```
cd ~/networknt/light-oauth2-config/dna
docker-compose -f docker-compose-oauth2-mysql.yml
```

Please verify the light-oauth2 serives are running.

### Set up SSO on Mac

The above light-oauth2 services are connecting to dna AD/KDC. In order to the browser application to login to light-oauth2 authorization code service to get authorization code redirect, we need to login to DNA from Keychain Access. 

1. Launch Keychain Access
2. From the Keychain Access menu, select ticket viewer.
3. Click add new identity and put your dna credential. 
4. Please make sure tha the userId is something like oauth2user1@DNA.EXAMLE.COM
5. Once log in successfully, a ticket will be issued with expiration time of 24h.
6. The next day, you can just click refersh button under the userId to renew a new ticket. 

### Set Hostname

For Kerberos/SPNEGO to work, the light-oauth2 server must use a DNS name match the SPN definition. Since we are using Mac Book Pro for our oauth2 server, we are goint to update /etc/hosts file to do a mapping. 

```
127.0.0.1       localhost dev.oauth.example.com
```

For official test environment or production, we should use DNS instead. 

### Test from a Browser. 

All major browsers are supporting Kerberos/SPNEGO these days. We are going to use Safari which has built in support for SPNEGO on Mac Book Pro. 

For browser setup please follow the links below. 

https://ping.force.com/Support/PingFederate/Integrations/How-to-configure-supported-browsers-for-Kerberos-NTLM

https://docs.spring.io/spring-security-kerberos/docs/current/reference/html/browserspnegoconfig.html

To test the code service with an default client, paste the following url in the address bar. 

```
https://dev.oauth.cibc.com:6881/oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization
```

If there is no popup window show up to ask your username and password and you have a redirect url returned with an authorization code, then that means the Kerberos/SPNEGO works. 


[default auth]: /service/oauth/authenticator/default-auth/

