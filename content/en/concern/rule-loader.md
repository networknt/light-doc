---
title: "Rule Loader"
date: 2021-11-22T22:14:38-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

In networknt, we have a rule engine [YAML Rule][] used for some repeatable business logic shared by multiple LOBs or organizations. 

To ensure multiple users can share the rules, we usually publish the rules to the light-portal, and applications can subscribe to rules created by other people. 

Once you have subscribed to some rules for your service, you should load those rules during your service startup. The rule-loader module has a startup hook implemented to load the rules for the service from the light portal. 

### Config

There is a config file called rule-loader.yml to specify the host of light-portal.

```
# rule loader config
# portal host
portalHost: ${ruleLoader.portalHost:https://localhost}

```

Other configuration files that are needed for this module is client.yml and client.truststore. 

[YAML Rule]: /library/yaml-rule/
