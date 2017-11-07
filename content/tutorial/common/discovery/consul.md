---
date: 2017-10-17T19:41:50-04:00
title: Consul service registry and discovery
---

Previous step multiple demonstrates how to use direct registry to enable load balance and
it works the same way as Consul and Zookeeper registry. In this step, we are going to
use Consul for registry to enable cluster.

Now let's copy from multiple to consul for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r multiple consul
cd ~/networknt/light-example-4j/discovery/api_b
cp -r multiple consul
cd ~/networknt/light-example-4j/discovery/api_c
cp -r multiple consul
cd ~/networknt/light-example-4j/discovery/api_d
cp -r multiple consul
```


### API A

In order to switch from direct registry to consul registry, we just need to update
service.yml configuration to inject the consul implementation to the registry interface.

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
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: localhost
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

Although in our case, there is no caller service for API A, we still need to register
it to consul by enable it in server.yml. Note that enableRegsitry is turn on and enableHttp
is turned off. 

If you don't turn off the http port, both ports will be registered on Consul. 

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
enableRegistry: true

```


### API B

Let's update service.yml to inject consul registry instead of direct registry used in
the previous step.

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
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: localhost
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

As API B will be called by API A, it needs to register itself to consul registry so
that API A can discover it through the same consul registry. To do that you only need
to enable server registry in config file.

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
enableRegistry: true
```

### API C

Although API C is not calling any other APIs, it needs to register itself to consul
so that API A can discovery it from the same consul registry. Let's create service.yml

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
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: localhost
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```


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
enableRegistry: true
```

### API D

Although API D is not calling any other APIs, it needs to register itself to consul
so that API B can discovery it from the same consul registry. Let's create service.yml

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
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: http
    - java.lang.String: localhost
    - int: 8500
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

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
httpsPort:  7444

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
enableRegistry: true

```


### Start Consul

Here we are starting consul server in docker to serve as a centralized registry. To make it
simpler, the ACL in consul is disable by setting acl_default_policy=allow.

```
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```


### Start four servers

Now let's start four terminals to start servers.  

API A

```
cd ~/networknt/light-example-4j/discovery/api_a/consul
mvn clean install -DskipTests
mvn exec:exec
```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/consul
mvn clean install -DskipTests
mvn exec:exec

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/consul
mvn clean install -DskipTests 
mvn exec:exec

```

API D


And start the first instance that listen to 7444 as default

```
cd ~/networknt/light-example-4j/discovery/api_d/consul
mvn clean install -DskipTests 
mvn exec:exec

```

If you follow the above sequence to start servers, then you will find that API A and 
API B show some error messages as they cannot establish connections to target servers. 

You can ignore these errors and as the connections will be recreated on the first request. 

Now you can see the registered service from Consul UI.

```
http://localhost:8500/ui
```


### Test four servers

```
curl -k https://localhost:7441/v1/data
```

And the result will be

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```
 
### Start another API D
 
Now let's start the second instance of API D. Before starting the serer, let's update
server.yml with port 7445.

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
enableRegistry: true

```

And start the second instance

```
cd ~/networknt/light-example-4j/discovery/api_d/consul
mvn clean install -DskipTests 
mvn exec:exec

```

### Test Servers

As we cache the connection in our application, so even there is a new server added to
consul, it won't be used unless we create a new connection for each request. 

Let's issue the same command again. 

```
curl -k https://localhost:7441/v1/data
```

And the result should be the same.

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

Shutdown the server with port 7444. Now when you call the same curl command, you will 
see 7445 server picks up the request.  

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7445","API D: Message 2 from port 7445","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```
