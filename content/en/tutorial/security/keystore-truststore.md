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
reviewed: true
---

The main difference between trustStore and keyStore is that trustStore (as name suggest) is used to store certificates from trusted Certificate authorities(CA) which are used to verify certificate presented by Server in SSL Connection, while keyStore is used to store private keys and identity certificates which programs should present to other parties (Server or client) to verify its identity. That was a one-liner difference between trustStore and keyStore in Java, but no doubt these two terms are quite confusing not just for anyone who is configuring SSL connection in Java for the first time, but also many intermediate and senior level programmers. One reason for this could be SSL setup being a one-time job, and not many programmers get the opportunity to do that. 

In this Java tutorial, we will explore both keyStore and trustStore and understand the main differences between them. By the way, you can use a keytool command to view certificates from trustStore and keyStore. The keytool command comes with Java installation and its available in the bin directory of JAVA_HOME.


### KeyStore vs TrustStore

The trustStore and keyStore are used in the context of setting up SSL connection in Java application between client and server. TrustStore and keyStore are very much similar in terms of construct and structure as both are managed by keytool command and represented by KeyStore programmatically but they often confuse Java programmers, both beginners and intermediate alike. The only difference between trustStore and keyStore is what they store, and their purpose. In SSL handshake, the purpose of trustStore is to verify credentials and purpose of keyStore is to provide the credential. The keyStore in Java stores private key and certificates corresponding to there public keys and require if you are SSL Server or SSL requires client authentication. TrustStore stores certificates from the third party, your Java application communicate or certificates signed by CA (certificate authorities like Verisign, Thawte, Geotrust or GoDaddy) which can be used to identify the third party. 

In order to understand the difference between keyStore and trustStore you need to understand how SSL conversation happens between client and server because this is the starting point of confusion. Many Java programmers don't pay attention to whether they are implementing the server side of SSL connection or client side of SSL Connection. 

For example,  setting up SSL for tomcat is server side of SSL while setting up JDBC over SSL is client side of SSL connection. If you are implementing SSL on the server side, you need a KeyStore to store your server certificate and private key. 

Anytime a client connects to the server, the server will present its certificate stored in KeyStore and client will verify that certificate by comparing with certificates stored on its trustStore.

Let's see the difference between trustStore and keyStore in point format which is much clear and easy to understand:

* Keystore is used to store your credential (server or client) while trustStore is used to store others credential (Certificates from CA).

* Keystore is needed when you are setting up server-side on SSL. It is used to store a server's identity certificate, which servers will present to a client on the connection while trustStore setup on client side must contain the server certificate to make the connection work. If your browser connects to any website over SSL, it verifies certificate presented by the server against its trustStore.

* Though I omitted this in the last section to reduce confusion you can have both keyStore and trustStore on client and server side if the client also needs to authenticate itself on the server. In this case, the client will store its private key and identify certificate on keyStore and server will authenticate the client against certificate stored on the server's trustStore.

* In Java -javax.net.ssl.keyStore property is used to specify keystore while -javax.net.ssl.trustStore is used to specify trustStore.

* In Java, one file can represent both keyStore and trustStore, but it's better to separate private and public credentials both for security and maintenance reasons.

* The trustStore and keyStore in Java 6 and newer releases. When you install JDK or JRE on your machine, Java comes with its own trustStore (collection of the certificate from well known CA like Verisign, goDaddy, thwarte, etc. You can find this file inside 
JAVA_HOME/JRE/Security/cacerts where JAVA_HOME is your JDK Installation directory. As the default cacerts trust most of the commercial root CA, so we do not use it but use externalized client.truststore. 

* keytool  command (binary comes with JDK installation inside JAVA_HOME/bin) can be used to create and view both keyStore and trustStore.

In summary, the keystore contains private keys and certificates used by SSL servers to authenticate themselves to SSL clients. By convention, such files are referred to as keystores.

When used as a truststore, the file contains certificates of trusted SSL servers, or of Certificate Authorities trusted to identify servers. There are no private keys in the truststore.

### Client and Server

Unlike the traditional web service, the client and server concept is a little bit complicated in microservices architecture. A server can act as a client to call another service, and at the same time, it is a server for its consumer. 

### One-Way SSL

In General, SSL authentication is used in websites to authenticate the server, which means that the server has to authenticate to the client by sending a digital certificate signed by a well known trusted CA. If the browser trusts the CA, the website is safe, and you will see the address bar highlighted in green.

In the above scenario, only the client authenticates the server to ensure that the server is trusted. This is called One-Way SSL. In microservices architecture, the client is not a browser but a service, and it must have a trust store with the server certificate in order to verify the server during the handshake. This file is called client.truststore in Light. To make things complicated, the client itself might be a server for other services. In this case, the same server instance will have server private key that is saved into a keystore. This file is called server.keystore in the Light platform. 

So in One-Way SSL, there are three scenarios and different keystore or truststore will be used. 

* An original client which only consumes service and itself is not serving anybody. In this instance, you just need client.truststore that saves all the certificates and chains from all the servers this client is about to connect to. A typical client is like a standalone application. 

* A server instance serves its client and calls other services. In this case, you need both server.keystore and client.truststore. The server.keystore has the private key for the server and client.truststore contains all the certificates and chains from the servers this instance is calling. This is a typical service in the middle of the chain pattern in microservices architecture.  

* A server only serves others and will never call another service. It is the last service in a chain pattern in microservices. In this case, this server will need server.keystore and it needs to give the public key certificates to all the consumers to put it into their client.truststore.
  
When you scaffold a project from light-codegen, the default settings are One-Way SSL. You can change to Two-Way SSL in the server.yml config file.
 
 
### Two-Way SSL

Two-Way SSL also called Mutual Authentication. In Mutual Authentication, in addition to server authentication, the client also has to present its certificate to the server. The server verifies it by checking if it is signed by a trusted CA and if it has tampered. If both server and client authenticated themselves, then SSL authentication is a success.

In an SSL Mutual Authentication scenario, these are the overall steps that take place during SSL handshake:

* Client initiates the request.
* The server sends its own certificate which is found from its server.keystore. 
* The client verifies its certificate if it can be trusted. If the server’s certificate or its CA’s certificate are found in client.truststore, then the server is authenticated.
* If client authentication is enabled at server side, the server requests for client’s certificate.
* The client sends its own certificate which is found from its client.keystore.
* The server verifies the client’s certificate if it can be trusted. If the client’s certificate or its CA’s certificate are found in its server.truststore, then the client is authenticated.

### Debugging SSL issues

To turn on SSL debugging, all you have to do is set the system property at startup javax.net.debug=all. This options can be added in the Java command line or as an option parameter in your IDE.

More information about SSL debugging is available here: http://docs.oracle.com/javase/1.5.0/docs/guide/security/jsse/ReadDebug.html

### Use keytool 

Keystore is used by a server to store private keys, and truststore is used by the third-party client to store public keys provided by the server.  Below are the steps for generating self-signed Java certificates for SSL communication:

* Generate a certificate using keygen command in windows:

keytool -genkey -keystore server.keystore -alias mycert-20161109 -keyalg RSA -keysize 2048 -validity 3950

* Self certify the certificate:

keytool -selfcert -alias mycert-20161109 -keystore server.keystore -validity 3950

* Export certificate to a folder:

keytool -export -alias mycert-20161109 -keystore server.keystore -rfc -file mycert-20161109.cer

* Import Certificate into client Truststore:

keytool -v -import -file ssg05.com.pem -alias somecrt -keystore client.truststore

* List keys in a keystore:

keytool -list -v -keystore client.truststore

For other command line options, please refer to [keytool][]

Here is an article that shows how to use keytool to generate self-signed certificates.

https://www.cloudera.com/documentation/enterprise/5-2-x/topics/cm_sg_create_key_trust.html

[keytool]: /tool/keytool/
