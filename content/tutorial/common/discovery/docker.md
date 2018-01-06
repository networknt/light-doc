---
date: 2017-10-17T21:05:20-04:00
title: Service discovery for Docker container
---

In this step, we are going to dockerize all the APIs and then use [registrator](https://github.com/gliderlabs/registrator) 
for service registry. To make it easier, we are going to use docker-compose to put everything together.

Now let's copy from consul to docker for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r consul consuldocker
cd ~/networknt/light-example-4j/discovery/api_b
cp -r consul consuldocker
cd ~/networknt/light-example-4j/discovery/api_c
cp -r consul consuldocker
cd ~/networknt/light-example-4j/discovery/api_d
cp -r consul consuldocker
```

### API A

As we will be using docker-compose to start consul, registrator and APIs altogether
and there is no easy way to control the start sequence. We are going to update the 
handler to remove the connection creation in constructor. 

```
package com.networknt.apia.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    static String apibHost;
    static String apicHost;
    static String path = "/v1/data";
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);

    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connectionB;
    static ClientConnection connectionC;

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        if(connectionB == null || !connectionB.isOpen()) {
            try {
                apibHost = cluster.serviceToUrl("https", "com.networknt.apib-1.0.0", null);
                connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        if(connectionC == null || !connectionC.isOpen()) {
            try {
                apicHost = cluster.serviceToUrl("https", "com.networknt.apic-1.0.0", null);
                connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final CountDownLatch latch = new CountDownLatch(2);
        final AtomicReference<ClientResponse> referenceB = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceC = new AtomicReference<>();
        try {
            ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestB, exchange);
            connectionB.sendRequest(requestB, client.createClientCallback(referenceB, latch));

            ClientRequest requestC = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestC, exchange);
            connectionC.sendRequest(requestB, client.createClientCallback(referenceC, latch));

            latch.await();

            int statusCodeB = referenceB.get().getResponseCode();
            if(statusCodeB >= 300){
                throw new Exception("Failed to call API B: " + statusCodeB);
            }
            List<String> apibList = Config.getInstance().getMapper().readValue(referenceB.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apibList);

            int statusCodeC = referenceC.get().getResponseCode();
            if(statusCodeC >= 300){
                throw new Exception("Failed to call API C: " + statusCodeC);
            }
            List<String> apicList = Config.getInstance().getMapper().readValue(referenceC.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apicList);
        } catch (Exception e) {
            logger.error("Exception:", e);
            throw new ClientException(e);
        }
        list.add("API A: Message 1");
        list.add("API A: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}

```

Since we are using registrator to register the service, we need to disable the application service registration.

server.yml

```

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  7001

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  7441

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
serviceId: com.networknt.apia-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false
```

Please note that enableRegistry is set to false as we don't need the server to register itself
as it is running inside the Docker container and doesn't know the Container IP address and port.

In order to discover services from client module, we need to update service.yml to setup properties
of URLImpl and properties of ConsulEcwidClient with hostname Consul and port 8500.

service.yml

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
  - com.networknt.consul.client.ConsulEcwidClient:
    - java.lang.String: consul
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

The Dockerfile generated from light-codegen should expose 7441 based on the config.json

Dockerfile

```
FROM openjdk:8-jre-alpine
EXPOSE  7441
ADD /target/apia-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```


Let's build a docker image for this service and push it to docker hub. For you own testing
you can ignore the last command to push the image to docker hub or you can change is to
your own docker hub account. 

```
cd ~/networknt/light-example-4j/discovery/api_a/consuldocker
mvn clean install -DskipTests
docker build -t networknt/com.networknt.apia-1.0.0 .
docker push networknt/com.networknt.apia-1.0.0
```

### API B

Similar with API A, we are going to remove the connection creation in constructor for
the handler. 

```
package com.networknt.apib.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    static String apidHost;
    static String path = "/v1/data";
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connection;

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        final CountDownLatch latch = new CountDownLatch(1);
        if(connection == null || !connection.isOpen()) {
            try {
                apidHost = cluster.serviceToUrl("https", "com.networknt.apid-1.0.0", null);
                connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(path);
            // this is to ask client module to pass through correlationId and traceabilityId as well as
            // getting access token from oauth2 server automatically and attatch authorization headers.
            if(securityEnabled) client.propagateHeaders(request, exchange);
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            if(statusCode >= 300){
                throw new Exception("Failed to call API D: " + statusCode);
            }
            List<String> apidList = Config.getInstance().getMapper().readValue(reference.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apidList);
        } catch (Exception e) {
            logger.error("Exception:", e);
            throw new ClientException(e);
        }
        list.add("API B: Message 1");
        list.add("API B: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}

```

server.yml

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  7002

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  7442

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
serviceId: com.networknt.apib-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

```

service.yml

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
  - com.networknt.consul.client.ConsulEcwidClient:
    - java.lang.String: consul
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

Dockerfile should be generated correctly based on the config.json for light-codegen


```
FROM openjdk:8-jre-alpine
EXPOSE  7442
ADD /target/apib-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build a docker image for this service and push it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_b/consuldocker
mvn clean install -DskipTests
docker build -t networknt/com.networknt.apib-1.0.0 .
docker push networknt/com.networknt.apib-1.0.0

```

### API C

server.yml

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  7003

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  7443

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
serviceId: com.networknt.apic-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false
```

service.yml

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
  - com.networknt.consul.client.ConsulEcwidClient:
    - java.lang.String: consul
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

Dockerfile should be

```
FROM openjdk:8-jre-alpine
EXPOSE  7443
ADD /target/apic-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

Let's build a docker image for this service and push it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_c/consuldocker
mvn clean install
docker build -t networknt/com.networknt.apic-1.0.0 .
docker push networknt/com.networknt.apic-1.0.0
```


### API D

server.yml

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

service.yml

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
  - com.networknt.consul.client.ConsulEcwidClient:
    - java.lang.String: consul
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

Dockerfile

```
FROM openjdk:8-jre-alpine
EXPOSE  7444
ADD /target/apid-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```


Let's build a docker image for this service and push it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_d/consuldocker
mvn clean install
docker build -t networknt/com.networknt.apid-1.0.0 .
docker push networknt/com.networknt.apid-1.0.0

```

### Start docker-compose

Now we have all four API built, dockerized and pushed to docker hub. Let's start the docker-compose.
Before we start the new docker-compose, make sure we stop the current consul docker container. 


```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-consul-registrator.yml up

```

This compose will start Consul, Registrator and four services together.


### Test Servers

```
curl -k https://localhost:7441/v1/data
```

And here is the result.

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```


Note: with the latest registrator, it registers apia, b, c and d into address 172.25.0.4 - 7
and these IPs are not reachable. It is broken and need further investigation. 

