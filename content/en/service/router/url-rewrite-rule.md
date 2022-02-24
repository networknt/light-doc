---
title: "URL Rewrite Rule"
date: 2022-02-24T17:04:28-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When we use the light-router as a gateway, the incoming URL might have a different path than the outgoing URL. To support it, you can add a section called urlRewriteRules in the router.yml to define a list of regex and replacement pairs. 

Here is an example of the configuration for the UNIT tests. 

```
# URL rewrite rules, each line will have two parts: the regex patten and replace string separated
# with a space. The light-router has service discovery for host routing, so whe working on the
# url rewrite rules, we only need to create about the path in the URL.
# Test your rules at https://www.freeformatter.com/java-regex-tester.html#ad-output
urlRewriteRules:
  # test your regex rule at https://www.freeformatter.com/java-regex-tester.html#ad-output
  # /listings/123 to /listing.html?listing=123
  - /listings/(.*)$ /listing.html?listing=$1
  # /ph/uat/de-asia-ekyc-service/v1 to /uat-de-asia-ekyc-service/v1
  - /ph/uat/de-asia-ekyc-service/v1 /uat-de-asia-ekyc-service/v1
  # /tutorial/linux/wordpress/file1 to /tutorial/linux/cms/file1.php
  - (/tutorial/.*)/wordpress/(\w+)\.?.*$ $1/cms/$2.php
  # used in the test case to router /v3/address request to /v2/address
  - /v3/(.*)$ /v2/$1

```

In the embedded router.yml, we have made this config property a template so that you can overwrite values in the values.yml file.


```
# URL rewrite rules, each line will have two parts: the regex patten and replace string separated
# with a space. The light-router has service discovery for host routing, so whe working on the
# url rewrite rules, we only need to create about the path in the URL.
# Test your rules at https://www.freeformatter.com/java-regex-tester.html#ad-output
urlRewriteRules: ${router.urlRewriteRules:}
```

The corresponding values.yml can have a section like the following. 

```
router.urlRewriteRules:
  # test your regex rule at https://www.freeformatter.com/java-regex-tester.html#ad-output
  # /listings/123 to /listing.html?listing=123
  - /listings/(.*)$ /listing.html?listing=$1
  # /ph/uat/de-asia-ekyc-service/v1 to /uat-de-asia-ekyc-service/v1
  - /ph/uat/de-asia-ekyc-service/v1 /uat-de-asia-ekyc-service/v1
  # /tutorial/linux/wordpress/file1 to /tutorial/linux/cms/file1.php
  - (/tutorial/.*)/wordpress/(\w+)\.?.*$ $1/cms/$2.php
  # used in the test case to router /v3/address request to /v2/address
  - /v3/(.*)$ /v2/$1


```

The rule is written as a string with two parts. The first part is the regex pattern and the second part is the replacement whent he pattern is matched. If one of the rules is matched in the list, the source and target path should be the same. 


The following is a very good online Java regex tester to verify your URL rewrite rules. Please be aware, we have other config files to handle the host routing so the URL rewrite rules will only apply to the path parameters. That is why you don't see the host in the above sample rules. 



