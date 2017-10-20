---
date: 2017-10-17T19:08:51-04:00
title: Multiple API D Instances
---

In this step, we are going to start two API D instances that listening to 7444 and 7445.

Now let's copy from dynamic to multiple for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r dynamic multiple
cd ~/networknt/light-example-4j/discovery/api_b
cp -r dynamic multiple
cd ~/networknt/light-example-4j/discovery/api_c
cp -r dynamic multiple
cd ~/networknt/light-example-4j/discovery/api_d
cp -r dynamic multiple
```

 
### API B 

Let's modify API B service.yml to have two API D instances that listen to 7444
and 7445. 

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apid-1.0.0: https://localhost:7444,https://localhost:7445
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

### API D

In order to start two instances with the same code base, we need to modify the
server.yml before starting the second server. 

Also, let's update the handler so that we know which port serves the request.

DataGetHandler.java

```
package com.networknt.apid.handler;

import com.networknt.config.Config;
import com.networknt.server.Server;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;

import java.util.ArrayList;
import java.util.List;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        int port = Server.config.getHttpsPort();
        List<String> messages = new ArrayList<>();
        messages.add("API D: Message 1 from port " + port);
        messages.add("API D: Message 2 from port " + port);
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}

```

### Start Servers
Now let's start all five servers from five terminals. API D has two instances.

API A

```
cd ~/networknt/light-example-4j/discovery/api_a/multiple
mvn clean install exec:exec
```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/multiple
mvn clean install exec:exec

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/multiple
mvn clean install exec:exec

```

API D


And start the first instance that listen to 7444.

```
cd ~/networknt/light-example-4j/discovery/api_d/multiple
mvn clean install exec:exec

```
 
Now let's start the second instance of API D. Before starting the serer, let's update
server.yml with https port 7445 and disable HTTP.

```

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  7004

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  7445

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled by default.
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: tls/server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: tls/server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.apid-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

```

And start the second instance

```
cd ~/networknt/light-example-4j/discovery/api_d/multiple
mvn clean install exec:exec

```

### Test Servers

```
curl http://localhost:7001/v1/data
```

And the result can be from port 7444 or 7445 as load balancer will pick up on of them.

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7445","API D: Message 2 from port 7445","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

If you send multiple requests, the result should be same as it is using the same cached connection.
Now let's kill the API D instance that serves the request (in our case 7445 instance) by CTRL+C and 
run the curl command again. You will see the following result as it failed over to the other server. 

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

Now, you can see API B can be failed over to either 7444 instance or 7445 instance of API D. 
As we are caching the connection in API B, we have to shutdown one instance in order to fail
over to another. You can change the API B to get a new connection for each request. This will
impact the performance however, it won't pin API B to once instance for ever. The server will
disconnect the connection if it is idle for a period time so eventually the load balancer will
put load equally to all instances. Developers can try to disconnect after a period of time or
a number of request to take advantage of cache and at the same time enable load balancing. 

The next step, we are going to use consul to do the service registry and discovery.
