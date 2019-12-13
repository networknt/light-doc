---
title: "Code Generation"
date: 2019-12-12T21:32:23-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we have created [specifications][] for four microservices. In the same folder along with the openapi.yaml and openapi.json, we also added two more files. README.md and config.json

In the README.md, there is a link to the original specification and more details about the API. In addition, there is a link and a command line to guide users to generate the project from the specification with light-codegen. 

config.json is a control file for light-codegen to direct code generation.

To learn how to use light-codegen, please visit [light-codegen tutorial][]. 

### accounts

The generated code can be found at https://github.com/open-banking/accounts/tree/master/generated

### balances

The generated code can be found at https://github.com/open-banking/balances/tree/master/generated

### parties

The generated code can be found at https://github.com/open-banking/parties/tree/master/generated

### transactions

The generated code can be found at https://github.com/open-banking/transactions/tree/master/generated


[specifications]: /tutorial/open-banking/spec/
[light-codegen tutorial]: /tutorial/generator/
