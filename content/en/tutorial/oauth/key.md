---
title: "Key Distribution"
date: 2017-11-10T14:51:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


Light-Java and Light-OAuth2 support distributed security verification and this
requires the JWT public key certificate to be distributed to all services. By
default, all services built on top of Light-Java will include a set of 
certificates. But how to distributed new certificates to thousands of running
services if certificates are renewed? There is no way we can copy certificates
to all the running containers as they are dynamic and new containers can be
started anytime by container orchestration tool. 

The traditional push approach is not working and a new way of pull certificates
from OAuth2 key service is implemented in Light-Java and Light-OAuth2.

This feature is tightly integrated with Light-Java and it should work seamlessly.

The first step to get certificate is to encode client_id:client_secret pair for
basic authentication. 

Here is the client_id:client_secret

```
f7d42348-c647-4efb-a52d-4c5787421e72:f6h1FTI8Q3-7UScPZDzfXA
```

Go to https://www.base64encode.org/ to encode it to

```
ZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyOmY2aDFGVEk4UTMtN1VTY1BaRHpmWEE=
```

To get certificate by a key id.

```
curl -k -H "Authorization: Basic ZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyOmY2aDFGVEk4UTMtN1VTY1BaRHpmWEE=" https://localhost:6886/oauth2/key/101
```

And here is the result.

```
-----BEGIN CERTIFICATE-----
MIIDkzCCAnugAwIBAgIEUBGbJDANBgkqhkiG9w0BAQsFADB6MQswCQYDVQQGEwJDQTEQMA4GA1UE
CBMHT250YXJpbzEQMA4GA1UEBxMHVG9yb250bzEmMCQGA1UEChMdTmV0d29yayBOZXcgVGVjaG5v
bG9naWVzIEluYy4xDDAKBgNVBAsTA0FQSTERMA8GA1UEAxMIU3RldmUgSHUwHhcNMTYwOTIyMjI1
OTIxWhcNMjYwODAxMjI1OTIxWjB6MQswCQYDVQQGEwJDQTEQMA4GA1UECBMHT250YXJpbzEQMA4G
A1UEBxMHVG9yb250bzEmMCQGA1UEChMdTmV0d29yayBOZXcgVGVjaG5vbG9naWVzIEluYy4xDDAK
BgNVBAsTA0FQSTERMA8GA1UEAxMIU3RldmUgSHUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCqYfarFwug2DwpG/mmcW77OluaHVNsKEVJ/BptLp5suJAH/Z70SS5pwM4x2QwMOVO2ke8U
rsAws8allxcuKXrbpVt4evpO1Ly2sFwqB1bjN3+VMp6wcT+tSjzYdVGFpQAYHpeA+OLuoHtQyfpB
0KCveTEe3KAG33zXDNfGKTGmupZ3ZfmBLINoey/X13rY71ITt67AY78VHUKb+D53MBahCcjJ9YpJ
UHG+Sd3d4oeXiQcqJCBCVpD97awWARf8WYRIgU1xfCe06wQ3CzH3+GyfozLeu76Ni5PwE1tm7Dhg
EDSSZo5khmzVzo4G0T2sOeshePc5weZBNRHdHlJA0L0fAgMBAAGjITAfMB0GA1UdDgQWBBT9rnek
spnrFus5wTszjdzYgKll9TANBgkqhkiG9w0BAQsFAAOCAQEAT8udTfUGBgeWbN6ZAXRI64VsSJj5
1sNUN1GPDADLxZF6jArKU7LjBNXn9bG5VjJqlx8hQ1SNvi/t7FqBRCUt/3MxDmGZrVZqLY1kZ2e7
x+5RykbspA8neEUtU8sOr/NP3O5jBjU77EVec9hNNT5zwKLevZNL/Q5mfHoc4GrIAolQvi/5fEqC
8OMdOIWS6sERgjaeI4tXxQtHDcMo5PeLW0/7t5sgEsadZ+pkdeEMVTmLfgf97bpNNI7KF5uEbYnQ
NpwCT+NNC5ACmJmKidrfW23kml1C7vr7YzTevw9QuH/hN8l/Rh0fr+iPEVpgN6Zv00ymoKGmjuuW
owVmdKg/0w==
-----END CERTIFICATE-----
```

