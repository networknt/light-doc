---
title: "Generate Service2"
date: 2019-04-16T08:17:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have created [service2 schema][] file and config.json file in the model-config repository. In this step, we are going to generate the service2 with light-codegen. 

For information on how to use light-codegen to scaffold hybrid service project, please visit [hybrid service][]

With both schema.json and config.json ready for the service2, let's run the following commands to scaffold the project. 

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/hybrid/hello-world/service2 -m model-config/hybrid/hello-world/service2/schema.json -c model-config/hybrid/hello-world/service2/config.json
```

After the generation command line is completed, a new project service2 should be generated in light-example-4j/hybrid/hello-world folder. 

Let's go to the service2 folder and build it. 

```
cd ~/networknt
cd light-example-4j/hybrid/hello-world/service2
./mvnw install
```

In the next step, we are going to update the service2 and [test it][] from the IntelliJ IDEA with unit/integration test cases.

[service2 schema]: /tutorial/hybrid/hello-world/service2-schema/
[hybrid service]: /tool/light-codegen/hybrid-service/
[test it]: /tutorial/hybrid/hello-world/test-service2/
