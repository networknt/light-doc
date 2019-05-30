---
title: "Register Handler"
date: 2019-05-30T09:30:46-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

As of light-4j 1.6.3 release, we have added a feature for light-hybrid-4j to allow individual handler to register as a serfvice on Consul so that client can discover it and invoke it directly. It gives the flexibility to deploy serverless services to any individual hybird server instance without making all server instance hosting the same number of services. 

In this tutorial, we are going to show you how to do that using light-codegen as an example. You can find the docker-compose and configuration files in light-config-test/light-codegen-handler folder. The purpose of this example is to allow different version of codegen-web handlers to be deployed and invoked to generate project based on different releases.

### Version Upgrade

In order to register handlers to the registry, all handlers must be versioned for each offcial release and snapshot release. The best way to do that is through the version-upgrade light-bot task. 

Here is the section in the version-upgrade.yml that will update all handlers' version for each release. 

```
  # update codegen web handler version and schema
  - path: codegen-web/src/main/java/com/networknt/codegen/handler/CodegenMultipleHandler.java
    match: id="lightapi.net/codegen/multiple/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/java/com/networknt/codegen/handler/CodegenSingleHandler.java
    match: id="lightapi.net/codegen/single/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/java/com/networknt/codegen/handler/FrameworkListHandler.java
    match: id="lightapi.net/codegen/listFramework/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/java/com/networknt/codegen/handler/SchemaGetHandler.java
    match: id="lightapi.net/codegen/getSchema/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/java/com/networknt/codegen/handler/ValidateUploadFileHandler.java
    match: id="lightapi.net/codegen/validateUploadFile/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
    
  - path: codegen-web/src/test/java/com/networknt/codegen/handler/FrameworkListHandlerTest.java
    match: ,\\"version\\":\\"((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)\\"
  - path: codegen-web/src/test/java/com/networknt/codegen/handler/GeneratorServiceHandlerTest.java
    match: \\"action\\":\\"single\\",\\"version\\":\\"((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)\\"
  - path: codegen-web/src/test/java/com/networknt/codegen/handler/GeneratorServiceHandlerTest.java
    match: \\"action\\":\\"multiple\\",\\"version\\":\\"((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)\\"
  - path: codegen-web/src/test/java/com/networknt/codegen/handler/GeneratorServiceHandlerTest.java
    match: "version", "((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/test/java/com/networknt/codegen/handler/SchemaGetHandlerTest.java
    match: ,\\"version\\":\\"((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)\\"

  - path: codegen-web/src/main/resources/schema.json
    match: "lightapi.net/codegen/listFramework/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/resources/schema.json
    match: "lightapi.net/codegen/validateUploadFile/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/resources/schema.json
    match: "lightapi.net/codegen/multiple/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"
  - path: codegen-web/src/main/resources/schema.json
    match: "lightapi.net/codegen/single/((([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?)"

```

The above is the added on section for the light-codegen. 

### Docker Compose

The docker-compose file can be found in the light-config-test/light-codegen-handler folder. Before starting this docker-compose, you need to start the consul docker-compose first locally. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

After you are sure that consul is up and running and can be accessed from localhost:8500 from the browser.

```
cd ~/networknt/light-config-test/light-codegen-handler/
docker-compose up -d
```

### Release/Snapshot

The configuraiton file are in both release folder and snapshot folder. 



