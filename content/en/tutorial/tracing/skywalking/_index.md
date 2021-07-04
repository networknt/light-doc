---
title: "Skywalking Tracing"
date: 2019-11-23T19:27:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

As of [Skywalking v6.5.0][1], there is support for light-4j (both versions 1.6.x and 2.x) in the form of a plugin. This page provides instructions on how to start generating traces on the Skywalking platform for a light-4j application.

### Obtaining Skywalking project component binaries

There are many components to the Skywalking project; the ones we are most interested in are: the Observability Analysis Platform (OAP) backend server, the Skywalking UI, and the Java agent which is to be installed to your application of interest.

For the OAP server and Skywalking UI, you have the choice of:

- building them from [source code][2]
- downloading the distributed [packages][3]
- running [Docker images][4]

The Java agent, named `skywalking-agent.jar`, must be either built from source or downloaded.

### In-depth workflow

This section provides an example workflow (suited for development) using macOS with detailed steps to set up Skywalking.

We will make use of the example Petstore project featured in the OpenAPI 3.0 [tutorial][5]. Note that you may need to modify the pom to have the `release` profile set as active by default. This is so that you can execute the `petstore-3.0.1.jar` with the `java -jar` command.

Follow the steps mentioned in the Skywalking *Build from GitHub* section [here][6]. In step 4, use the tag `v6.5.0` or higher. Upon a successful build, there should be a `~/$WORKSPACE/skywalking/skywalking-agent/skywalking-agent.jar` produced.

Run `git clone https://github.com/apache/skywalking-docker.git`. Navigate to the directory that contains the Skywalking Docker repo that you cloned and then run `cd ~/$WORKSPACE/skywalking-docker/6/6.5/compose`. Once inside that folder, run `docker-compose up`, which will download the OAP server and Skywalking UI images if needed and then start the containers.

*Alternatively*, you can run the OAP server and Skywalking UI locally. When the Skywalking project is built successfully, there is a `dist` folder which contains `.tar.gz` (for *nix) and `.zip` files (for Windows). Extract the appropriate archive and in the `bin` folder you will see several `.sh` (for *nix) and `.bat` (for Windows) files. To start the OAP server, run the `oapService` script. To start the Skywalking UI, run the  `webappService.sh` script. Note that each script starts a new process, so once finished you will have to search for the `pid`s and then kill them. More details including configuration and advanced features are available [here][7]. 

Navigate back to the Petstore project and then run `java -javaagent:$WORKSPACE/skywalking/skywalking-agent/skywalking-agent.jar -jar target/petstore-3.0.1.jar`.

*Alternatively*, you may run `java -javaagent:$WORKSPACE/skywalking/skywalking-agent/skywalking-agent.jar -Dskywalking.plugin.light4j.trace_handler_chain=true -jar target/petstore-3.0.1.jar`. By specifying `plugin.light4j.trace_handler_chain=true`, traces generated upon sending a request to the service endpoint will include spans for whichever light-4j handlers were involved with the request. Detailed information on setting up the Java agent can be found [here][8].

Now make some requests to the Petstore service with curl:

```
curl -k https://localhost:8443/v1/pets
curl -k https://localhost:8443/v1/pets/1
curl -k https://localhost:8443/abc
```

When you view the Skywalking UI (located at http://localhost:8080 by default), you should see something like the following for a successful request:

![Example Skywalking UI trace screen](/images/skywalking_ui_trace.png)

Note that Skywalking also has a plugin for Undertow, which is why you can see span information related to Undertow in the light-4j trace from the screenshot.

For HTTP 4xx errors, you will instead see something like:

![Example Skywalking UI trace error](/images/skywalking_ui_error.png)

Services built with light-4j are shown in the topology as such:

![Skywalking UI topology](/images/skywalking_ui_topology.png)

### Troubleshooting

Here are some tips that may be useful:

- Check that the timezone displayed in the Skywalking UI (bottom right corner) matches the OAP backend server and is the one you expect; traces will not show up immediately otherwise

[1]: https://github.com/apache/skywalking/tree/v6.5.0
[2]: https://github.com/apache/skywalking/blob/master/docs/en/guides/How-to-build.md
[3]: http://skywalking.apache.org/downloads/
[4]: https://github.com/apache/skywalking-docker
[5]: https://www.networknt.com/tutorial/rest/openapi/petstore/
[6]: https://github.com/apache/skywalking/blob/master/docs/en/guides/How-to-build.md#build-from-github
[7]: https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/backend-ui-setup.md
[8]: https://github.com/apache/skywalking/blob/master/docs/en/setup/service-agent/java-agent/README.md

