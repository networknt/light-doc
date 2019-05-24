---
title: "mask.yml"
date: 2019-01-09T10:24:06-05:00
description: ""
categories: [configs]
keywords: []
aliases: []
toc: false
draft: false
---
mask.yml is the configuration file for masking logging info.  
this configuration file can also be used for centralizing mask rules.  
[dump handler](/concern/dump) and [audit handler](/concern/dump) will use this config to mask logging info based on those **predefined fields**, those fields will be shown in below.

| Field Name |        Type                             | Description |
|:----------:|:---------------------------------------:|:-----------:|
|   string   | [MaskString Object](#MaskString Object) |             |
|    regex   |  [MaskRegex Object](#MaskRegex Object)  |             |
|    json    |  [MaskJson Object](#MaskJson Object)   |             |
   
#### <a id="MaskString Object">MaskString Object</a>
| Field Name |       Type       |                                      Description                                     |
|:----------:|:----------------:|:------------------------------------------------------------------------------------:|
|    uri     |        map(${Pattern}: ${Replacement})       | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)** <br/>**mask url by trying to match child keys(patterns), <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> uri:<br/>&nbsp;&nbsp;&nbsp;&nbsp; password=[^&]*: password=****** <br/>&nbsp;&nbsp;&nbsp;&nbsp; number=\d{1,16}: number=----------------|
|    ${key}  |        map(${Pattern}: ${Replacement})       |                                                                                      |
**Usage:**<br/>
Mask a String by matching each child key(pattern) defined in ${key}

```
String url1 = "/v1/customer?sin=123456789&password=secret&number=1234567890123456";
String output = Mask.maskString(url1, "uri");
Assert.assertEquals("/v1/customer?sin=masked&password=******&number=----------------", output);
```
#### <a id="MaskRegex Object">MaskRegex Object</a>
|    Field Name   | Type |                                        Description                                       |
|:---------------:|:----------------------------------:|:----------------------------------------------------------------------------------------:|
|  queryParameter |  map(${ParameterName}: ${Pattern}) | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)**  <br/>**mask query parameters based on parameter name defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> queryParameter:<br/>&nbsp;&nbsp;&nbsp;&nbsp; accountNo: "(.*)"|
|  requestHeader  |  map(${headerName}: ${Pattern})    | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)**  <br/>**mask request headers based on header name defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> requestHeader:<br/>&nbsp;&nbsp;&nbsp;&nbsp; header1: "(.*)"|
|  responseHeader |  map(${headerName}: ${Pattern})    | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)**  <br/>**mask response headers based on header name defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> responseHeader:<br/>&nbsp;&nbsp;&nbsp;&nbsp; header1: "(.*)"|
|  requestCookies |  map(${cookieName}: ${Pattern})    | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)**  <br/>**mask request cookies based on header name defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> requestCookies:<br/>&nbsp;&nbsp;&nbsp;&nbsp; cookie1: "(.*)"|
| responseCookies |  map(${cookieName}: ${Pattern})    | **Predefined field for [AuditHandler](/concern/audit) and [DumpHandler](/concern/dump)**  <br/>**mask response cookies based on header name defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> responseCookies:<br/>&nbsp;&nbsp;&nbsp;&nbsp; cookie2: "(.*)"|
|     ${set}      |  map(${key}: ${Pattern})           | |

**Usage:**<br/>
Mask a String based on key:  
1. find the ${key} defined in ${set}  
2. try to match the string with ${Pattern}  
3. replace the match with "*"  
```
String testHeader2 = "tests";
String output2 = Mask.maskRegex(testHeader2, "requestHeader", "header2");
Assert.assertEquals(output2, "*****");
```

#### <a id="MaskJson Object">MaskJson Object</a>
|    Field Name   | Type |                                        Description                                       |
|:---------------:|:----------------------------------:|:----------------------------------------------------------------------------------------:|
|  requestBody   |  map(${JsonPath}: ${Pattern})    | **Predefined field for [DumpHandler](/concern/dump)**  <br/>**mask request body json based on json path defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> requestBody:<br/>&nbsp;&nbsp;&nbsp;&nbsp; "$.*.email": "(.\*)"|
|  responseBody  |  map(${JsonPath}: ${Pattern})    | **Predefined field for [DumpHandler](/concern/dump)**  <br/>**mask response body json based on json path defined inside, <br/>then replace the matches with values defined** <br />example:<br/> in mask.yml defines as following:<br/> responseBody:<br/>&nbsp;&nbsp;&nbsp;&nbsp; "$.product[\*].item[\*].name": "(.*)"|

**Usage:**<br/>
Mask.maskJson(inputJsonStr, "test2")
Config in mask.yml:
```
test2:
    "$.list.*.name": "(.*)"
    "$.list.*.accounts.*": "(.*)"
    "$.list1": "(.*)"
    "$.password": "(.*)"
```
Input Json:
```
{"name":"Steve",
"list":[
    {"name": "Nick"},
    {
        "name": "Wen",
        "accounts": ["1", "2", "3"]
    },
    {
        "name": "Steve",
        "accounts": ["4", "5", "666666"]
    },
    "secret1", "secret2"],
"list1": ["1", "333", "55555"],
"password":"secret"}
```
Output Json
```
{"name":"Steve","list":[{"name":"****"},{"name":"***","accounts":["*","*","*"]},{"name":"*****","accounts":["*","*","******"]},"secret1","secret2"],"list1":["*","***","*****"],"password":"******"}
```
#### An Example of mask.yml
```
description: mask configuration for different type of inputs
   string:
     uri:
   #    password=[^&]*: password=******
   #    number=\d{1,16}: number=----------------
   #    sin=\d{1,9}: sin=masked
   regex:
     queryParameter:
   #    accountNo: "(.*)"
     requestHeader:
   #    header1: "(.*)"
   #    header2: "(.*)"
     responseHeader:
   #    header3: "(.*)"
     requestCookies:
   #    userName: "(.*)"
     responseCookies:
   #    sensitiveData: "(.*)"
   json:
     requestBody:
   #    "$.*.email": "(.*)"
   #    "$.product[*].item[*].name": "(.*)"
     responseBody:
   #    "$.product[*].item[*].name": "(.*)"
   #    "$.product[*].item[*].name[0]": "(.*)"
   ```