---
title: Public Https
linktitle: Including Client Dependencies 
date: 2018-06-06T23:15:01-04:00
weight: 100
sections_weight: 100
draft: false
toc: false
---

When using our Http2Client to communication with public API with CA-signed certificates, we have to find a way to download the certificate from the site and put it into our client.trustore file in order to establish TLS connection to the site. 

There are multiple options to do that and the simple one is from the browser; however, the certificate downloaded from the browser might not be completed as we need to the entire chain in order to verify the certificate during the handshake. 

Here is a command line to do that.


```
openssl s_client -showcerts -connect us-east-1.online.tableau.com:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >tableau.pem

```

Once you have downloaded the certificate, you can test it with wget command line. 


```
wget https:/us-east-1.online.tableau.com --ca-certificate=tableau.pem

```

Now you can import it into the client.truststore file.

```
keytool -v -import -file tableau.pem -alias tableaucrt -keystore client.truststore 

```

For other keytool commands and options, please refer to [keytool][] document.

After the client.truststore is modified, you need to use Http2Client to access the target server. There are numeric examples that can be found in light-example-4j and all the test cases. 

Please note that you need to set the host header for all the requests. 

```
request.getRequestHeaders().put(Headers.HOST, "localhost");

```

To see the demo application source code which accesses tableau server, please refer to https://github.com/networknt/light-example-4j/tree/master/client/tableau

[keytool]: /tool/keytool/

