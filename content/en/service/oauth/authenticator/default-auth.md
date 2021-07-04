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
reviewed: true
---

The default authentication/authorization provider covers most use cases for most organizations. If there are slight differences in your use case, you can customize it or create another auth provider based on the default one.  

It is implemented in light-oauth2/authhub/src/main/java/com/networknt/oauth/DefaultAutenticator class. There is also DefaultAuth marker interface to assist service module loading for Singleton factory. 


### Authentication

The authentication logic is very simple, it first checks if the passed in Credential is an instance of LightPasswordCredential. If yes, then it tries to match the password with the users' cache which is mapped to the user service of light-oauth2. 

If the passed in Credential is an instance of LightGSSContextCredential, then it has been authenticated by Kerberos/SPNEGO. 

### Authorization

Once the authentication is done, an account object will be created based on the credential. An additional roles object needs to be created. For the LightPasswordCredential, the roles column of the user table will be used. For the LightGSSContextCredential, it will call LdapUtil.authorize(id) to get the roles from LDAP and assign it to the account. 


### Kerberos/SPNEGO KDC Configuration

There are a lot of online articles about how Kerberos/SPNEGO KDC setup on Microsoft platform. In our unit test case, we are using an embedded Apache Directory server for LDAP, KDC and Active Directory services. The configuration for these services can be found in the test folder of the code module. 

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

In the above command line, the SPN is HTTP/dev.oauth.example.com@DNA.EXAMPLE.COM, and the keytab file name is krb_oauth2code-allenc.keytab

The admin who helps to set up the KDC needs to provide client certificate for the KDC as well. Normally, there would be a root certificate and an issuing certificate. 

- Create several test users

If the server is not the production, several test user accounts need to be created. For production, it is not necessary as all employees must be created already. Some special groups might be added to certain users for testing purpose. 

### Light-oauth2 Configuration

As user authentication is part of the authorization code grant flow, all the authenticator configuration will be externalized for the code service. Even with all the information provided, it is hard for most people to config the code service with Kerberos/SPNEGO SSO. To help users to learn it, we have put an example config folder at https://github.com/networknt/light-config-test/tree/master/light-oauth2/dna-spnego

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

This file is for Kerberos configuration. In development mode, it should be located in light-oauth2/code/src/test/resources folder so that it can be automatically loaded. For docker or jar deployment, it should be located in the externalized config folder like other config files. 

Here is the Dockerfile for code server. 

```
FROM openjdk:8-jre-alpine
ADD /target/oauth2-code.jar server.jar
CMD ["/bin/sh","-c","java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -Djava.security.krb5.conf=/config/krb5.conf -jar /server.jar"]

```

As you can see we have an extra option -Djava.security.krb5.conf to specify what to load the krb5.conf in the command line. 

The following is an example for the krb5.conf config file. 

```
[libdefaults]
	default_realm = DNA.EXAMPLE.COM
	default_tgs_enctypes = des-cbc-md5,des3-cbc-sha1-kd,arcfour-hmac-md5
	default_tkt_enctypes = des-cbc-md5,des3-cbc-sha1-kd,arcfour-hmac-md5
	kdc_timeout = 5000
	dns_lookup_realm = false
	dns_lookup_kdc = false
	allow_weak_crypto = yes
	forwardable = true

[realms]
	DNA.EXAMPLE.COM = {
		kdc = dna.example.com:88
		default_domain = dna.example.com
	}

[domain-realm]
.dna.example.com = DNA.EXAMPLE.COM
dna.example.com = DNA.EXAMPLE.COM

[appdefaults]
pam = {
	debug = true
	ticket_lifetime = 600
	renew_lifetime = 600
	forwardable = true
	krb4_convert = false
}

[login]
	krb4_convert = true
	krb4_get_tickets = false

```

- secret.yml

For LDAP authentication/authorization, there is a function id create with a password. In order to leverage password encryption automatically, we put the password into the secret.yml file as following.

```
# SPNEGO/Kerberos SPN password
spnegoServicePassword: p0COauth2
```

- spnego.yml

This file is the configuration for SPNEGO which is used by the KerberosKDCUtil in authhub module. Here is an example. 

```
# If SPNEGO/Kerberos debugging is enabled. Should be disabled for production. 
debug: true
# If keytab file is used. If Microsoft AD is used, then set it to true
useKeyTab: true
# The keytab file location. It should be in the /config folder for Kubernetes
keyTab: /config/krb_oauth2code_poc-allenc.keytab
# The SPN principal name, normally it will be HTTP/{hostname}@domain
principal: HTTP/dev.oauth.example.com@DNA.EXAMPLE.COM

```

- client.truststore

You will receive one or two certificates from KDC admin, and you need to import them into the client.truststore for code service. Normally, if your KDC uses a self-signed certificate, then there should only be one. Otherwise, there might be one root certificate and one issuing certificate. 

For information on how to use the keytool to import certs, please refer to [keystore truststore][] and [keytool][].

- keytab file

Once the SPN is created on the MS domain controller, a keytab file will be created. The filename might be different for each deployment and it should be specified in spnego.yml config file keyTab entry. In the above spnego.yml file, the keyTab is specified at /config/krb_oauth2code_poc-allenc.keytab which means the same config folder like other config files. 



[keystore truststore]: /tutorial/security/keystore-truststore/
[keytool]: /tool/keytool/

