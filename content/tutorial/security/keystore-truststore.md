---
title: "Keystore Truststore"
date: 2017-12-13T20:50:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Main difference between trustStore vs keyStore is that trustStore (as name suggest) is used to store 
certificates from trusted Certificate authorities(CA) which are used to verify certificate presented 
by Server in SSL Connection while keyStore is used to store private key and own identity certificate 
which program should present to other parties (Server or client) to verify its identity. That was one 
liner difference between trustStore vs keyStore in Java but no doubt these two terms are quite a confusion 
not just for anyone who is the first time doing SSL connection in Java but also many intermediate and 
senior level programmer. One reason of this could be SSL setup being a one-time job and not many 
programmers get opportunity to do that. 

In this Java tutorial, we will explore both keystore and truststore and understand key differences 
between them. By the way, you can use a keytool command to view certificates from truststore and 
keystore. keytool command comes with Java installation and its available in the bin directory of JAVA_HOME.


### KeyStore vs TrustStore

In order to understand the difference between keyStore and trustStore you need to understand How SSL 
conversation happens between client and server because this is the starting point of confusion, many 
Java programmer doesn't pay attention whether they are implementing the server side of SSL connection 
or client side of SSL Connection. 

One example is setting up SSL for tomcat is server side of SSL while setting up JDBC over SSL is client 
side of SSL connection. If you are implementing SSL on Server side you need a KeyStore to store your server 
certificate and private key. 

Anytime a client will connect to the server, server will present its certificate stored in KeyStore and 
client will verify that certificate by comparing with certificates stored on its trustStore.

Let's see difference between truststore vs keystore in point format which is much clear and easy to 
understand:

1) Keystore is used to store your credential (server or client) while truststore is used to store others 
credential (Certificates from CA).

2) Keystore is needed when you are setting up server side on SSL, it is used to store server's identity 
certificate, which server will present to a client on the connection while truststore setup on client side 
must contain to make the connection work. If you browser to connect to any website over SSL it verifies 
certificate presented by server against its truststore.

3) Though I omitted this on the last section to reduce confusion but you can have both keystore and 
truststore on client and server side if the client also needs to authenticate itself on the server. 
In this case, client will store its private key and identify certificate on keystore and server will 
authenticate the client against certificate stored on server's truststore.

4) In Java -javax.net.ssl.keyStore property is used to specify keystore while -javax.net.ssl.trustStore 
is used to specify trustStore.

5) In Java, one file can represent both keystore vs truststore but it's better to separate private and 
public credential both for security and maintenance reason.

trusstore vs keystore in Java6) When you install JDK or JRE on your machine, Java comes with its own 
truststore (collection of certificate from well known CA like Verisign, goDaddy, thwarte etc. you can 
find this file inside

JAVA_HOME/JRE/Security/cacerts where JAVA_HOME is your JDK Installation directory.

7) keytool  command (binary comes with JDK installation inside JAVA_HOME/bin) can be used to create and 
view both keyStore and trustStore.

In summary, the keystore contains private keys and certificates used by SSL servers to authenticate 
themselves to SSL clients. By convention, such files are referred to as keystores.

When used as a truststore, the file contains certificates of trusted SSL servers, or of Certificate 
Authorities trusted to identify servers. There are no private keys in the truststore.

### Client and Server



### One-Way SSL

In General, SSL authentication is used in websites to authenticate the server. Which means that the server 
has to authenticate to the client by sending a digital certificate signed by a well known trusted CA. If the 
CA is trusted by the browser, the website is safe and you will see the address bar highlighed in green.

In the above scenario, only client authenticate the server to ensure that the server is trusted. This is call
One-Way SSL. In microservices architecture, the client is no a browser but a service and it must have a trust
store with the server certificate in order to verify the server during hand shake. This file is called
client.truststore in light platform. To make things complicated, the client itself might be a server for
other services. In this case, the same server instance will have server private key that is saved into
a keystore. This file is called server.keystore in light platform. 

So in One-Way SSL, there are three scenarios and different keystore or truststore will be used. 

1. Original client which only consumes service and itself is not serving anybody. On this instance, you
only need client.truststore that saves all the certificates and chains from all the servers this client
is about to connect to. A typical client is like a standalone application. 

2. A server instance sever its client and calls other services. In this case, you need both server.keystore
and client.truststore. The server.keystore has the prviate key for the server and client.truststore contains
all the certificates and chains from the servers this instance is calling. This is a typical service in
the middle of the chain pattern in microservices architecture.  

3. A server only serve others and will never call other service which is the last service in a chain
pattern in microservices. In this case, this server will only need server.keystore and it needs to give
the public key certificates to all the consumers to put it into their client.truststore.
  
When you scaffold a project from light-codegen, the default settings is One-Way SSL. You can change to 
Two-Way SSL in the server.yml config file.
 
 
### Two-Way SSL

Two-Way SSL also called Mutual Authentication. In Mutual Authentication, in addition to server authentication, 
the client also has to present its certificate to the server. The server verifies it by checking if it is 
signed by a trusted CA and if it is tampered. If both server and client authenticated themselves, then SSL 
authentication is a success.

In a SSL Mutual Authentication scenario, these are the overall steps that take place during SSL handshake:

* Client initiates the request.
* The server sends its own certificate which is found from its server.keystore. 
* The client verifies its certificate if it can be trusted. If the server’s certificate or its CA’s certificate are found in client.truststore, then the server is authenticated.
* If client authentication is enabled at server side, the server requests’s for client’s certificate.
* The client sends its own certificate which is found from its client.keystore.
* The server verifies the client’s certificate if it can be trusted. If the client’s certificate or its CA’s certificate are found in its client.truststore, then the client is authenticated.


### Debugging SSL issues

To turn on SSL debugging, all you had to do is set the system property at startup javax.net.debug=all.

More information about SSL debugging is available here: http://docs.oracle.com/javase/1.5.0/docs/guide/security/jsse/ReadDebug.html

### Use keytool 

Keystore is used by a server to store private keys, and truststore is used by third party client to store public keys provided by server to access. I have did that in my production application. Below are the steps for generating java certificates for SSL communication:

1. Generate a certificate using keygen command in windows:

keytool -genkey -keystore server.keystore -alias mycert-20161109 -keyalg RSA -keysize 2048 -validity 3950

2. Self certify the certificate:

keytool -selfcert -alias mycert-20161109 -keystore server.keystore -validity 3950

3. Export certificate to folder:

keytool -export -alias mycert-20161109 -keystore server.keystore -rfc -file mycert-20161109.cer

4. Import Certificate into client Truststore:

keytool -v -import -file ssg05.com.pem -alias somecrt -keystore client.truststore

5. List keys in keystore:

keytool -list -v -keystore client.truststore


Here is an article that shows how to use keytool to generate self-signed certificates.

https://www.cloudera.com/documentation/enterprise/5-2-x/topics/cm_sg_create_key_trust.html

 