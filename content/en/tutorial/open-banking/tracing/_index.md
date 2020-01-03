---
title: "Tracing with Jaeger"
date: 2020-01-01T08:34:31-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [proxy][] step, we have set up a light-proxy instance to mimic the transactions service in case transactions service is not built on top of light-4j and doesn't have any cross-cutting concerns addressed. 

Given there are several distributed services in the same application, monitoring these services on production will be a challenge. In this section, we are going to explore the distributed tracing with Jaeger. We also support Skywalking tracer from Apache and will implement it if someone interested. 

The first step is to copy the proxy folder to the tracing folder in the light-config-test folder in open-banking as our base configuration. Then we make the following updates. 

The detailed tutorial for the Jaeger can be found at [Jaeger Tutorial][]. 

### Start the Jaeger

Add the following service to the docker-compose. 

```
    jaeger:
      image: jaegertracing/all-in-one:1.9
      environment: 
        - COLLECTOR_ZIPKIN_HTTP_PORT=9411
      network_mode: host

```

With the above addition, the jaeger server will be started in the docker-compose. 

### Update services

The jaeger module from light-4j is not included in the pom.xml in the generated projects. For all the projects, we need to add this dependency to the pom.xml file. Let's create a folder called tracing and copy the code from security.

First for each services in the tracing folder, add the following to the pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

After make the change for pom.xml for accounts, balances, parties and transactions. Rebuild the docker images with the following commands for each. 

```
cd ~/open-banking/accounts/tracing
build.sh 1.0.0
```

### Update Configurations

Let's update the configurations for these services in the light-config-test repository. 

First, we need to add jaeger middleware handler to the handler.yml file in the handler chain. We also comment out the traceability and correlation handlers in the chain. As handler.yml wasn't externalized before, we need to copy it from each project to the corresponding folder. 

```
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger

```

In the default chain section.

```
    # - traceability
    # - correlation
    - jaeger
```

In the service.yml, we need to add a StartupHookProvider to initialize the Jaeger client. 

```
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  # If you are using mask module to remove sensitive info before logging, uncomment the following line.
  - com.networknt.jaeger.tracing.JaegerStartupHookProvider

```

Last, we need to add jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

Checking the config files and go to the test2 server to restart the docker-compose. As we have updated the docker images, we need to remove the images in order to download the latest. 


To access the UI of Jaeger on test2 server, we need to enable the port with the firewall. 

```
sudo ufw allow 16686
```

As accoutns is calling the balances and transactions, we need to ensure that the client.yml is updated with `injectOpenTracing: true`. Let's upgrade the values.yml with the following. 

```
openapi-security.enableVerifyJwt: true
server.dynamicPort: true
server.enableRegistry: true
client.verifyHostname: false
client.injectOpenTracing: true
```


[proxy]: /tutorial/open-banking/proxy/
[Jaeger Tutorial]: /tutorial/tracing/jaeger/
