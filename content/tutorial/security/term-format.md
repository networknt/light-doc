---
title: "Basic Terminologies and Formats of Certs and Keys"
date: 2018-10-31T08:58:55-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

There’s some confusion for the light-4j users about how to control which certificates are used for one-way TLS and two-way TLS. In this section, we will clear up the general terminology and formats about the certificate and private key to make it easier to understand other tutorials. 

### Basic Terminology

- CERTIFICATE OR CERT

The public half of a public/private key pair, though it’s not generally referred to as a key. This part is freely given to anyone, and it can be downloaded from the site you are accessing.

- PRIVATE KEY

A private key is never given out publicly. It is used to sign or encrypt data. A private key can be used to verify that its corresponding certificate was used to sign or encrypt things and vice versa.

- CERTIFICATE SIGNING REQUEST OR CSR

A file that you generate with your private key. You can send just the CSR to your CA, and they will create a signed certificate for you.

- CERTIFICATE AUTHORITY OR CA

These are places like Thawte that you pay in order to get a certificate that browsers will accept. You can also use someone like letsencrypt.org to get a free certificate that browsers will accept. You can also generate your own simple CA using openssl. A CA uses its private key to digitally sign a CSR and create a signed cert so that browsers can use the CA’s cert to tell that your cert is approved by that CA.

- DISTINGUISHED NAME OR DN

It’s a grouping of RDNs (Relative Distinguished Names). An RDN is something like “CN=your name”. This one means that the Common Name is set to the string “your name”. A DN would be something like “CN=your name,OU=Engineering,O=Netorknt”. In this case, OU means Organizational Unit and O means Organization.

- X509

A specification governing the format and usage of certificates.

### Format of certificate and key

There are many different formats used for storing keys and certs on-disk, but the most common ones are probably PEM, PKCS12, and JKS. The formats are not treated equally by Java, so it’s important to understand the different formats. It’s not obvious how to manipulate these formats, so I’ve also included sample commands for working with them.

- PEM

PEM is just DER that’s been Base64 encoded. It looks like this for a certificate:

```
-----BEGIN CERTIFICATE-----
(base 64 encoded stuff)
-----END CERTIFICATE-----

```

or for a private key:

```
-----BEGIN PRIVATE KEY-----
(base 64 encoded stuff)
-----END PRIVATE KEY-----
```

Sometimes PEM files will have a human-readable block of text above the Base64 encoded block. You can safely remove this human-readable text. Some applications prefer the cert PEM and the private key PEM to be in one file. (Apache httpd is one of them if I remember correctly.) Since PEM files are just plain text, you can do that with cat: 

```
cat cert.pem key.pem > cert-with-key.pem
```

While you could arbitrarily combine as many PEM blocks as you wanted into one file, typically they are kept separate except for this one case. You can get a human-readable description of a certificate in PEM format with 

```
openssl x509 -in cert.pem -noout -text
```
Certs are X509 formatted, hence the ‘x509′ subcommand to openssl. For keys, the command is 

```
openssl rsa -in key.pem -text -noout
```

Private keys can also be encrypted, in which case the marker block will say BEGIN ENCRYPTED PRIVATE KEY. You can create the decrypted form of the key with 

```
openssl rsa -in key-encrypted.pem -out key-decrypted.pem
```

- PKCS12

PKCS12 is a password-protected format that can contain multiple certificates and keys.

You can view the contents of a PKCS12 file (typically .p12 is used for PKCS12 files) with `openssl pkcs12 -in file.p12`. Add `-info` for a little bit more metadata. Note that if the file includes a private key, openssl will ask you for another password after asking for the decryption password for the PKCS12 file. This second password is used to encrypt the private key before displaying its PEM data to you. You could put this data in a separate file and decrypt it as shown above if you want the decrypted form.

You can create PKCS12 files with or without private keys or CA certs.

Cert and key: 
```
openssl pkcs12 -export -out cert-and-key.p12 -in cert.pem -inkey key.pem
```
Cert and key that includes the CA cert that signed the cert: 
```
openssl pkcs12 -export -out cert-and-key-with-ca.p12 -in cert.pem -inkey key.pem -CAfile /path/to/cacert.pem -chain
```
Cert without key (useful for CA certs): 
```
openssl pkcs12 -export -out cacert.p12 -in cacert.pem -nokeys
```

- JKS

A JKS keystore stores multiple certs and keys like PKCS12, but it’s just a Java specific, not a widespread standard like PKCS12. The tool to manage JKS files is ‘keytool’ which ships with the JDK. Entries in a JKS file have an “alias” that must be unique. If you don’t specify an alias, it will use “mycert” by default. This is fine if you’re only putting one cert in a keystore, but if you add another cert you’ll get an error because it will try to use the same default alias twice. JKS keystores also have a password, just like PKCS12. You can use keytool to add PEM and PKCS12 files.

Pleae refer to [keytool] for more information on the command line.

[keytool]: /tool/keytool/