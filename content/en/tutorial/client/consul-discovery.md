---
title: "Consul Discovery"
date: 2018-10-03T18:40:48-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

We have an issue opened by one of our users who is using Weblogic 12c in Java 8 to call microservices built top of the light-rest-4j framework in OpenShift cluster. The services are deployed with host networking, and IPs and port numbers are registered to the Consul cluster running outside of the OpenShift cluster. 

It is a typical deployment, and we have a lot of similar configurations with other users. The issue they were facing is that the first time lookup takes up to 10 minutes sometimes. The details can be found in this [issue][]

In order to reproduce the issue in my Kubernetes cluster, I am going to deploy the [service discovery tutorial][] to my cluster and build a client application to do the lookup periodically. I thought this is a very good tutorial for others to learn how to use Consul discovery in a standalone client, so I have written down the steps below. 

Here are services registered on the Consul. We have 4 services that form a chain pattern, and each service has 6 replicas. 

![consul services](/images/consul-services.png)

Click the apia service, then you can see all 6 healthy instances and their IPs and port numbers. 

![apia instances](/images/apia-instances.png)

Now we can pick one instance up and test it with curl commmand line. 

```
curl -k https://38.113.162.53:2407/v1/data
```

Result

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Now we can confirm that all the services are working together. The next step, we are going to create a standalone Java client which calls the apia using consul discovery. 

As we are going to use real Consul cluster for service discovery, we are going to copy the standalone client project to consul folder in light-example-4j/client. 

```
cd ~/networknt
cd light-example-4j/client
cp -r standalone consul
```

Now let's remove the main class and create a new class called Http2ClientConsul.java

```
package com.networknt.client;

import com.networknt.cluster.Cluster;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.xnio.IoUtils;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

/**
 * This is a stand along java client application that is used to reproduce one of the
 * issues opened by users. This client will connect to the Kubernetes cluster and
 * Consul cluster for service discovery. The business logic is very similar with the
 * standalone example but the configuration would be different.
 *
 * As the focus is to test Consul service discovery, OAuth is disabled here.
 *
 * @author Steve Hu
 */
public class Http2ClientConsul {
    // Get the singleton Http2Client instance/home/steve/networknt/light-example-4j/discovery/api_a/http-health/src/main/resources/config/client.truststore
    static Http2Client client = Http2Client.getInstance();

    public static void main(String[] args) throws Exception {
        Http2ClientConsul e = new Http2ClientConsul();
        e.testHttp2Get();
        System.exit(0);
    }

    /**
     * This is a simple example that create a new HTTP 2.0 connection for get request
     * and close the connection after the call is done. As you can see that it is using
     * a hard coded uri which points to an statically deployed service on fixed ip and port
     *
     * @throws Exception
     */
    public void testHttp2Get() throws Exception {
        // Get the singleton Cluster instance
        Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
        // Do the service discovery for every call to reproduce the issue
        String apiaHost = cluster.serviceToUrl("https", "com.networknt.apia-1.0.0", null, null);
        // Create one CountDownLatch that will be reset in the callback function
        final CountDownLatch latch = new CountDownLatch(1);
        // Create an HTTP 2.0 connection to the server
        final ClientConnection connection = client.connect(new URI(apiaHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        // Create an AtomicReference object to receive ClientResponse from callback function
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            final ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath("/v1/data");
            request.getRequestHeaders().put(Headers.HOST, "localhost");
            // send request to server with a callback function provided by Http2Client
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            // wait for 100 millisecond to timeout the request.
            latch.await(1000, TimeUnit.MILLISECONDS);
        } finally {
            // here the connection is closed after one request. It should be used for in frequent
            // request as creating a new connection is costly with TLS handshake and ALPN.
            IoUtils.safeClose(connection);
        }
        int statusCode = reference.get().getResponseCode();
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        System.out.println("testHttp2Get: statusCode = " + statusCode + " body = " + body);
    }

}

```

As you can see, we do the lookup once and call the service apia once. This is to test the water to ensure that your application is working. Then we can loop it periodically. 

The following config files need to be updated to make the client work. But first thing first, we need to replace the default client.truststore with the one in light-example-4j/discovery/api_a/http-health/src/main/resources/config folder. That file contains the certificate from Consul server. Please refer to the [discovery tutorial][] for more info on prepare the client.truststore.

consul.yml is needed to specify the consulUrl which is pointing to one of the nodes on our Consul cluster. As we do not register the client as a service, other properties in this file are not used. 

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: true
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false

```

secret.yml contains a consul token which grant the access to the Consul cluster.

```
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527

# EmailSender password
emailPassword: change-to-real-password

```

service.yml defines the configuration for service discovery components. 

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: consul
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

Now let's run the application in the IDE and verify the print. And we got something like this. 


```
testHttp2Get: statusCode = 200 body = ["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]

```

This means the client is working. Now let's update the main class to add a loop and call the discovery and send a request every 5 seconds for 1000 times. 

```
    public static void main(String[] args) throws Exception {
        Http2ClientConsul e = new Http2ClientConsul();
        for(int i = 0; i < 1000; i++) {
            e.testHttp2Get();
            Thread.sleep(5000);
        }
        System.exit(0);
    }
```
 
Now the client is running, let's wait and see if we can reproduce the issue.

After a long time, the test completed without any issue. We have to go back to our user to ask help to reproduce it. 

Stay tuned. 


[issue]: https://github.com/networknt/light-4j/issues/303
[service discovery tutorial]: /tutorial/common/discovery/http-health/
[discovery tutorial]: /tutorial/common/discovery/consul-tls/
