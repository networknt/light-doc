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
reviewed: true
---

In the previous [proxy][] step, we have set up a light-proxy instance to mimic the transactions service in case transactions service is not built on top of light-4j and doesn't have any cross-cutting concerns addressed. 

Given there are several distributed services in the same application, monitoring these services on production will be a challenge. In this section, we are going to explore the distributed tracing with Jaeger. We will also support Skywalking tracer from Apache and will implement it if someone interested. 

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

The other way is to start the Jaeger with a standalone docker container. 

```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.9
```

### Update services

The jaeger module from light-4j is not included in the pom.xml in the generated projects. For all the projects, we need to add this dependency to the pom.xml file. Let's create a folder called tracing and copy the code from security.

First for each service in the tracing folder, add the following to the pom.xml

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

After making the change for pom.xml for accounts, balances, parties, and transactions, rebuild the docker images with the following commands for each. 

```
cd ~/open-banking/accounts/tracing
./build.sh 1.0.0
```

### Update Configurations

Let's update the configurations for these services in the light-config-test repository. 

First, we need to add the jaeger middleware handler to the handler.yml file in the handler chain. We also comment out the traceability and correlation handlers in the chain. As handler.yml wasn't externalized before, we need to copy it from each project to the corresponding folder. 


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


To access the UI of Jaeger on the test2 server, we need to enable the port with the firewall. 

```
sudo ufw allow 16686
```

As accounts service is calling the balances and transactions, we need to ensure that the client.yml is updated with `injectOpenTracing: true`. Let's upgrade the values.yml with the following. 

```
openapi-security.enableVerifyJwt: true
server.dynamicPort: true
server.enableRegistry: true
client.verifyHostname: false
client.injectOpenTracing: true
```

Now, let's issue a request from the command line. 

```
curl -k https://ob.lightapi.net/accounts/22289 \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'
```

After you receive the response, you can go to the Jaeger UI to check how many services are involved in the request and how much time each service spent. 

For the test cloud, the Jaeger server can be access from your browser with the following address. 

```
http://38.113.162.52:16686/
```

In the next step, we are going to build a single page application [client][] to access the APIs and deploy it on the router instance on the portal server as a virtual host. In this setup, there is no need to enable CORS to access the APIs from the SPA. 

{{< youtube yWF3QIc69XU >}}

[proxy]: /tutorial/open-banking/proxy/
[Jaeger Tutorial]: /tutorial/tracing/jaeger/
[client]: /tutorial/open-banking/client/
