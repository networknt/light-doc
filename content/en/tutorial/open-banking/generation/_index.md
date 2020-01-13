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
reviewed: true
---

In the previous step, we have created [specifications][] for four microservices. In the same folder, along with the `openapi.yaml` and `openapi.json`, we also added two more files. README.md and config.json

In the README.md, there is a link to the original specification and more details about the API. In addition, there is a link and a command line to guide users to generate the project from the specification with light-codegen. 

config.json is a control file for light-codegen to direct code generation.

There are three main ways to utilize the light-codegen to scaffold a new project. 

If you are a casual developer and don't want to build the light-codegen project locally, you can utilize the https://codegen.lightapi.net to generate the project in a zip file. 

If you are working on more projects at the same time, build the light-codegen locally and use the command line is faster and more productive. 

If you are working on a specification that is modified very often and need to generate the project repeatedly, using the light-codegen maven plugin is more convenient. 

To learn how to use light-codegen, please visit [light-codegen tutorial][]. 

### accounts

The generated code can be found at https://github.com/open-banking/accounts/tree/master/generated

### balances

The generated code can be found at https://github.com/open-banking/balances/tree/master/generated

### parties

The generated code can be found at https://github.com/open-banking/parties/tree/master/generated

### transactions

The generated code can be found at https://github.com/open-banking/transactions/tree/master/generated

The above four generated projects can be built and started with the following command. 

```
mvn clean install exec:exec
```

You can send requests to these services, but there is no response at the moment as the generated handlers return an empty string as the response. 

[specifications]: /tutorial/open-banking/spec/
[light-codegen tutorial]: /tutorial/generator/
