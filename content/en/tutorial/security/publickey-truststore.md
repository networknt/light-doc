---
title: "Adding server certificate to client.truststore"
date: 2018-02-23T15:35:03-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the [keystore and truststore tutorial][], we have discussed the difference of between keystore and truststore and in which case, you need to update the following files to enable one-way TLS or two-way TLS. 

* client.keystore - client private key.
* client.truststore - public key certificates the client can trust. 
* server.keystore - server private key.
* server.truststore - public key certificates the server can trust.

Also, there are another two files related to the certificates in oauth config folder. 

* primary.crt - the primary public key certificate that is used to verify JWT token signature.
* secondary.crt - the secondary public key certificate that is used to verify JWT token signature.

If you are running light-oauth2 server, then you need the following files in oauth config folder.

* primary.jks - the primary private key to sign the signature of JWT token.
* secondary.jks - the secondary private key to sign the signature of JWT token.


In the following section, we are going to walk through a most common scenario for service building. The process is based on a real project that we need to enable light-proxy to talk to a backend service built on top of Neo4j graph database. 

Once Neo4j server is up and running with self-signed TLS certificate, we get a file called public.crt from the Neo4j admin. Now we need to import the public.crt into client.truststore in config folder for light-proxy so that light-proxy can establish TLS handshake with Neo4j server.


Out of the box, light-platform provides a client.truststore that contains the public key certificate for the private key in server.keystore. This enables TLS out of the box right after the project scaffolding. In the default configuration for the light-proxy test environment, there is an existing client.truststore already. The following steps will import the public.crt and remove the old public key certificate from the client.truststore.


```
keytool -import -trustcacerts -alias neo4j -file public.crt -keystore client.truststore
```

Above command assumes that public.crt and client.truststore are in the same folder. When you run the command, you will be asked for a password. The default password for the client.truststore is password and you should change it and modify secret.yml file with the changed password. Also, you
can even [encrypt][] the sensitive data in secret.yml file. 

If neo4j alias has been imported before, you will have an error message like this. 

```
keytool error: java.lang.Exception: Certificate not imported, alias <neo4j> already exists
```

If that is the case, let's remove the old certificate from client.truststore and run the import
command again. 

```
keytool -delete -alias neo4j -keystore client.truststore
```

Now we have the public.crt imported into the client.truststore and the next step is to remove the
existing certificate in the default client.truststore. This is an important step for production
and light-platform has a [certification process][] that can run again production service to ensure that default certificate won't go to production. 

 
Before remove the old certificate, let's see how many certificates in the file. 

```
keytool -list -v -keystore client.truststore
```

You will find there is one with alias name "server" in the truststore. Let's remove it with the
following command line. 

```
keytool -delete -alias server -keystore client.truststore
``` 

You can use -list command to double check if the alias server is removed.

Now you can use the client.truststore to connect to neo4j backend with https enabled. 


[keystore and truststore tutorial]: /tutorial/security/keystore-truststore/
[encrypt]: /tutorial/security/encrypt-decrypt/
[certification process]: https://github.com/networknt/light-portal/tree/master/api-certification
