---
title: "Default Auth"
date: 2018-07-27T11:46:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The default authentication/authorization provider covers most use cases for most organizations. If there are slight differences in your use case, you can customize it or create another auth provider based on the default one.  

It is implemented in light-oauth2/authhub/src/main/java/com/networknt/oauth/DefaultAutenticator class. There is also DefaultAuth marker interface to assist service module loading for Singleton factory. 


### Authentication

The authentication logic is very simple, it first check if the passed in Credential is an instance of LightPasswordCredential. If yes, then try to match the password with the users cache which is mapped to the user service of light-oauth2. 

If the passed in Credential is an instance of LightGSSContextCredential, then it has been authenticated by Kerberos/SPNEGO. 

### Authorization

Once the authentication is done, an account object will be created based on the credential. An additional roles object needs to be created. For the LightPasswordCredential, the roles column of user table will be used. For the LightGSSContextCredential, it will call LdapUtil.authorize(id) to get the roles from LDAP and assign it to the account. 


### Kerberos/SPNEGO KDC Configuration

There are a lot of online articles about how Kerberos/SPNEGO KDC setup on Microsoft platform. In our unit test case, we are using an embedded Apache Directory server for LDAP, KDC and Active Directory services. The configuration for these services can be found in the test folder of code module. 

When setup up Microsoft AD, please refer to its document. However, the following outlines the major steps. 

- create a function id on AD 

This is for light-oauth2 to connect to LDAP to get groups after authentication. For example to create an account krb_oauth2code with a password like p0COauth2.

For the function id, open account "Properties", in "Account" tab, check "User cannot change password" and "Password never expires" but uncheck any other options. Switch to "Delegation" tab, select "Trust this user for delegation to any service(Kerberos only)". 

- create SPN and keytab file

Log in with fire ID for AD domain controller and create a c:\temp directory if not already existing. 

For example the following command line is used to create a POC SPN and keytab. 
```
ktpass -princ HTTP/dev.oauth.example.com@DNA.EXAMPLE.COM -crypto All -ptype KRB5_NT_PRINCIPAL -mapuser dna\krb_oauth2code -kvno 0 -pass p0COauth2 -out C:\temp\krb_oauth2code-allenc.keytab
```

In the above command line, the SPN is HTTP/dev.oauth.example.com@DNA.EXAMPLE.COM and the keytab file name is krb_oauth2code-allenc.keytab

The admin who helps to setup the KDC needs to provide client certificate for the KDC as well. Normally, there would be a root certificate and an issuring certificate. 

- Create several test users

If the server is not the production, several test user accounts need to be created. For production, it is not necessary as all employees must be created already. Some special groups might be added to certain users for testing purpose. 

### Light-oauth2 Configuration

As user authentication is part of the authorization code grant flow, all the authenticator configuration will be externalized for the code service. 

The following files are listed here as examples. 

- service.yml

Please ensure that all used auth providers are wired in. Here is an example that has two auth providers. 

```
- com.networknt.oauth.auth.Authenticator<com.networknt.oauth.auth.DefaultAuth>:
  - com.networknt.oauth.auth.DefaultAuthenticator
- com.networknt.oauth.auth.Authenticator<com.networknt.oauth.auth.MarketPlaceAuth>:
  - com.networknt.oauth.auth.MarketPlaceAuthenticator

```


- krb5.conf

This file is for Kerberos configuration. In development mode, it should be located in light-oauth2/code/


