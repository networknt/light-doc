---
title: "SalesforceHandler"
date: 2022-04-27T13:31:00-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Some organizations use cloud services like Salesforce cloud service with the light-gateway as the external services gateway. The light-gateway will be responsible for addressing all the cross-cutting concerns like security, tracing, metrics, auditing, etc. 

Salesforce has a special authentication flow to get an access token, so we have implemented a customized SalesforceHandler to handle the authentication and invoke the real request to the target server.


Here is the detailed step for the flow. 


* Generate a key pair and public key certificate.
* Your organization must be on board with Salesforce and send your public key certificate to Salesforce. 
* Before sending a real request to Salesforce, we need to create a JWT token in the handler and sign it with the private key. 
* Send the JWT token to the token URL defined in the config with urn:ietf:params:oauth:grant-type:jwt-bearer
* The response will have an access token, scope and instance_url. 
* Create a request to the serviceHost in the config with the token in the Authorization header. 
* Return the response to the caller in the handler.


For organizations using the Mcaffee gateway for Internet access, you need to add the proxyHost and proxyPort in your config. 

To sign the JWT token locally, you need to have a pfx or jks file that contains the private key. The corresponding public key certificate should be sent to Salesforce. For example, a file named apigatewayuat.pfx in the resources/config folder. The private key alias is apigatewayuat, which is the filename without the extension. 

We also need to create client.truststore with the public key certificate to connect to the Salesforce site. I have downloaded the entire chain from the https://test.salesforce.com from the browser. 


I have a sample client.truststore in src/test/resources/config folder with the following certificates. The password is `password`. It has the cert to test.salesforce.com and the root certificate. 

```
steve@lucky:~/networknt/light-4j/ingress-proxy/src/test/resources/config$ keytool -v -list -keystore client.truststore
Enter keystore password:  
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 3 entries

Alias name: digicert global root ca
Creation date: Apr. 22, 2022
Entry type: trustedCertEntry

Owner: CN=DigiCert TLS RSA SHA256 2020 CA1, O=DigiCert Inc, C=US
Issuer: CN=DigiCert Global Root CA, OU=www.digicert.com, O=DigiCert Inc, C=US
Serial number: 6d8d904d5584346f68a2fa754227ec4
Valid from: Tue Apr 13 20:00:00 EDT 2021 until: Sun Apr 13 19:59:59 EDT 2031
Certificate fingerprints:
	 SHA1: 1C:58:A3:A8:51:8E:87:59:BF:07:5B:76:B7:50:D4:F2:DF:26:4F:CD
	 SHA256: 52:27:4C:57:CE:4D:EE:3B:49:DB:7A:7F:F7:08:C0:40:F7:71:89:8B:3B:E8:87:25:A8:6F:B4:43:01:82:FE:14
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
AuthorityInfoAccess [
  [
   accessMethod: ocsp
   accessLocation: URIName: http://ocsp.digicert.com
, 
   accessMethod: caIssuers
   accessLocation: URIName: http://cacerts.digicert.com/DigiCertGlobalRootCA.crt
]
]

#2: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: 03 DE 50 35 56 D1 4C BB   66 F0 A3 E2 1B 1B C3 97  ..P5V.L.f.......
0010: B2 3D D1 55                                        .=.U
]
]

#3: ObjectId: 2.5.29.19 Criticality=true
BasicConstraints:[
  CA:true
  PathLen:0
]

#4: ObjectId: 2.5.29.31 Criticality=false
CRLDistributionPoints [
  [DistributionPoint:
     [URIName: http://crl3.digicert.com/DigiCertGlobalRootCA.crl]
]]

#5: ObjectId: 2.5.29.32 Criticality=false
CertificatePolicies [
  [CertificatePolicyId: [2.16.840.1.114412.2.1]
[]  ]
  [CertificatePolicyId: [2.23.140.1.1]
[]  ]
  [CertificatePolicyId: [2.23.140.1.2.1]
[]  ]
  [CertificatePolicyId: [2.23.140.1.2.2]
[]  ]
  [CertificatePolicyId: [2.23.140.1.2.3]
[]  ]
]

#6: ObjectId: 2.5.29.37 Criticality=false
ExtendedKeyUsages [
  serverAuth
  clientAuth
]

#7: ObjectId: 2.5.29.15 Criticality=true
KeyUsage [
  DigitalSignature
  Key_CertSign
  Crl_Sign
]

#8: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: B7 6B A2 EA A8 AA 84 8C   79 EA B4 DA 0F 98 B2 C5  .k......y.......
0010: 95 76 B9 F4                                        .v..
]
]



*******************************************
*******************************************


Alias name: server
Creation date: Jan. 19, 2013
Entry type: trustedCertEntry

Owner: CN=localhost, OU=OU, O=Org, L=City, ST=State, C=GB
Issuer: CN=localhost, OU=OU, O=Org, L=City, ST=State, C=GB
Serial number: 50faad68
Valid from: Sat Jan 19 09:27:52 EST 2013 until: Tue Jan 17 09:27:52 EST 2023
Certificate fingerprints:
	 SHA1: 87:3B:92:F5:0A:FE:99:E9:C5:AF:38:F1:C8:65:98:7A:C1:13:19:1D
	 SHA256: 13:F6:78:18:41:3D:7D:39:82:7F:1C:1A:13:BD:48:57:DB:DF:E1:F4:FA:8C:8E:6D:BE:43:2B:64:C0:F3:D7:20
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3


*******************************************
*******************************************


Alias name: test.salesforce.com
Creation date: Apr. 21, 2022
Entry type: trustedCertEntry

Owner: CN=test.salesforce.com, O="salesforce.com, inc.", L=San Francisco, ST=California, C=US
Issuer: CN=DigiCert TLS RSA SHA256 2020 CA1, O=DigiCert Inc, C=US
Serial number: 23e1abf53b37b132874802338a8b8fd
Valid from: Wed Jul 28 20:00:00 EDT 2021 until: Thu Jul 28 19:59:59 EDT 2022
Certificate fingerprints:
	 SHA1: F4:BD:3F:05:4A:0B:83:7C:E5:A4:D5:DE:32:EC:9B:7E:33:5E:0F:62
	 SHA256: 0A:CE:53:48:76:9B:2D:1C:14:61:89:4F:B1:48:9B:07:FD:38:03:34:52:1E:5A:B8:46:24:5E:72:C0:58:18:69
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 1.3.6.1.4.1.11129.2.4.2 Criticality=false
0000: 04 82 01 6A 01 68 00 76   00 29 79 BE F0 9E 39 39  ...j.h.v.)y...99
0010: 21 F0 56 73 9F 63 A5 77   E5 BE 57 7D 9C 60 0A F8  !.Vs.c.w..W..`..
0020: F9 4D 5D 26 5C 25 5D C7   84 00 00 01 7A EF A4 3E  .M]&\%].....z..>
0030: 93 00 00 04 03 00 47 30   45 02 20 21 D8 82 D5 02  ......G0E. !....
0040: 3C BC 23 3A BA DE F3 36   B3 48 C1 95 93 AB 37 76  <.#:...6.H....7v
0050: 26 B6 56 F7 D8 B6 FB 8E   D0 4A 7F 02 21 00 B0 75  &.V......J..!..u
0060: EF 5F 9A 11 1F D7 E6 28   4F A2 75 D5 3D FF 7E DE  ._.....(O.u.=...
0070: 92 68 DF 21 FF 8D F1 64   8B 2C 2D 30 6A 87 00 75  .h.!...d.,-0j..u
0080: 00 41 C8 CA B1 DF 22 46   4A 10 C6 A1 3A 09 42 87  .A...."FJ...:.B.
0090: 5E 4E 31 8B 1B 03 EB EB   4B C7 68 F0 90 62 96 06  ^N1.....K.h..b..
00A0: F6 00 00 01 7A EF A4 3E   A0 00 00 04 03 00 46 30  ....z..>......F0
00B0: 44 02 20 64 57 19 AA BA   1E 4E F6 36 79 5B 57 8F  D. dW....N.6y[W.
00C0: AF 4B B2 EE 27 63 68 4F   81 30 C2 F8 A7 78 84 DD  .K..'chO.0...x..
00D0: 9B B0 40 02 20 78 21 20   C6 00 19 56 A4 57 66 52  ..@. x! ...V.WfR
00E0: E0 BA D8 3D 05 EE 9C DB   2E AE DA D4 6D E2 64 4B  ...=........m.dK
00F0: 82 A9 0B 3D CE 00 77 00   DF A5 5E AB 68 82 4F 1F  ...=..w...^.h.O.
0100: 6C AD EE B8 5F 4E 3E 5A   EA CD A2 12 A4 6A 5E 8E  l..._N>Z.....j^.
0110: 3B 12 C0 20 44 5C 2A 73   00 00 01 7A EF A4 3E F3  ;.. D\*s...z..>.
0120: 00 00 04 03 00 48 30 46   02 21 00 CA 44 F5 55 5E  .....H0F.!..D.U^
0130: 68 D7 5C 2B 8F C4 6A 7F   CB C1 77 31 E8 61 59 A2  h.\+..j...w1.aY.
0140: AC D0 8D C0 D8 3A 8A 54   A3 33 C4 02 21 00 EF EA  .....:.T.3..!...
0150: AD 84 7C 8E 19 EE 86 CA   72 5A 19 E3 F2 E9 9F AF  ........rZ......
0160: EB 5D F2 57 A4 BD 7B 69   19 DA 24 8B FC 0A        .].W...i..$...


#2: ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
AuthorityInfoAccess [
  [
   accessMethod: ocsp
   accessLocation: URIName: http://ocsp.digicert.com
, 
   accessMethod: caIssuers
   accessLocation: URIName: http://cacerts.digicert.com/DigiCertTLSRSASHA2562020CA1-1.crt
]
]

#3: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: B7 6B A2 EA A8 AA 84 8C   79 EA B4 DA 0F 98 B2 C5  .k......y.......
0010: 95 76 B9 F4                                        .v..
]
]

#4: ObjectId: 2.5.29.19 Criticality=true
BasicConstraints:[
  CA:false
  PathLen: undefined
]

#5: ObjectId: 2.5.29.31 Criticality=false
CRLDistributionPoints [
  [DistributionPoint:
     [URIName: http://crl3.digicert.com/DigiCertTLSRSASHA2562020CA1-1.crl]
, DistributionPoint:
     [URIName: http://crl4.digicert.com/DigiCertTLSRSASHA2562020CA1-1.crl]
]]

#6: ObjectId: 2.5.29.32 Criticality=false
CertificatePolicies [
  [CertificatePolicyId: [2.23.140.1.2.2]
[PolicyQualifierInfo: [
  qualifierID: 1.3.6.1.5.5.7.2.1
  qualifier: 0000: 16 1B 68 74 74 70 3A 2F   2F 77 77 77 2E 64 69 67  ..http://www.dig
0010: 69 63 65 72 74 2E 63 6F   6D 2F 43 50 53           icert.com/CPS

]]  ]
]

#7: ObjectId: 2.5.29.37 Criticality=false
ExtendedKeyUsages [
  serverAuth
  clientAuth
]

#8: ObjectId: 2.5.29.15 Criticality=true
KeyUsage [
  DigitalSignature
  Key_Encipherment
]

#9: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: test.salesforce.com
  DNSName: test.database.com
  DNSName: test.cloudforce.com
]

#10: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 58 B3 80 48 FA 53 73 83   0A 3A 01 E8 5D D5 1B FE  X..H.Ss..:..]...
0010: 4C A2 20 31                                        L. 1
]
]



*******************************************
*******************************************

```

Here is the config file salesforce.yml

```
# Configuration for SalesforceHandler
# Indicate if the handler is enabled or not
enabled: ${salesforce.enabled:false}
# Salesforce get jwt token URL to send the request
tokenUrl: ${salesforce.tokenUrl:https://test.salesforce.com/services/oauth2/token}
# Authentication issuer
authIssuer: ${salesforce.authIssuer:2MVG9CM7abZT_gV7nAVssYIKEY2otVSwr.I4itTgn6mvS9xedke}
# Authentication subject
authSubject: ${salesforce.authSubject:conquestintegration@networknt.com}
# Authentication audience
authAudience: ${salesforce.authAudience:https://test.salesforce.com}
# Certificate file name. The private key alias is the filename without the extension.
certFilename: ${salesforce.certFilename:apigatewayuat.pfx}
# Certificate file password
certPassword: ${salesforce.certPassword:password}
# IV
iv: ${salesforce.iv:YoeAb3a/Epqoge}
# Token time to live
tokenTtl: ${salesforce.tokenTtl:60}
# Wait length
waitLength: ${salesforce.waitLength:5000}
# Proxy Host if calling within the corp network with a gateway like Mcafee gateway.
proxyHost: ${salesforce.proxyHost:}
# Proxy Port if proxy host is used. default value will be 443 which means HTTPS.
proxyPort: ${salesforce.proxyPort:}
# If HTTP2 is used to connect to the salesforce site.
enableHttp2: ${salesforce.enableHttp2:true}
# A list of applied request path prefixes, other requests will skip this handler. The value can be a string
# if there is only one request path prefix needs this handler. or a list of strings if there are multiple.
appliedPathPrefixes: ${salesforce.appliedPathPrefixes:}
# Salesforce target service host for service access with the token get with above property.
serviceHost: ${salesforce.serviceHost}

```

To overwrite some of the properties, we have the following values.yml in the src/test/resources/config for test cases. Please note that some of the values are changed to remove the identity information. 

```
# handler.yml
handler.handlers:
  .
  .
  .
  # add the SalesforceHandler to the end of the handler list.
  - com.networknt.proxy.SalesforceHandler@salesforce

handler.chains.default:
  .
  .
  .
  - header
  # add the saleforce handler after the header and before the prefix and router.
  - saleforce
  - prefix
  - router
  .
  .
  .

# client.yml
client.verifyHostname: false

# salesforce.yml
salesforce.enabled: true
salesforce.authIssuer: 3MVG9SUKGR3O.vJebzXvWgAEZ9KX_7vmKt6k3g_AdsTWtzBTkwU9jTTJF9h5mL.2tJaPW
salesforce.authSubject: apiuser.conquestintegration@networknt.sit
salesforce.certFilename: apigatewayuat.pfx
salesforce.certPassword: S1dsfereae!gtdwd3vu4degte
salesforce.iv: YeoadsfcAvs2a/PeereEaRqg
salesforce.proxyHost: gateway.example.com
salesforce.proxyPort: 443
salesforce.appliedPathPrefixes:
  - /salesforce
  - /services/apexrest
salesforce.serviceHost: https://nnt-sit.my.salesforce.com

```



For developers, you can use the test case to ensure everything works.

The body of the request after remove the identity information. 

```
{"clientId":"FSC_0aere014cA2","loggedInUserEmail":"steve.hu@networknt.com"}
```

The post request URL is something like this.

```
/services/apexrest/NNT_ConquestApplicationServices/getAccountDetailsForConquestToUpdatePlan
```

The response should be a JSON without the details.

```
{"errorReturnCode":200,"errorMessage":"Successful","ConquestJSON":"{......}}
```

