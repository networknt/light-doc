---
date: 2017-10-17T21:03:06-04:00
title: Multiple instances with tags per environment
---

In the previous step, we've setup the services to register on Consul and discover
downstream services through Consul. If there are multiple instances for the same
serviceId, then the client will try to load balance these instances.

In real service delivery, there are a lot complicated testing scenarios that need
to be addressed and one of the use cases is [environment segregation](../../../design/env-segregation.md)

In this step, we are going to use environment tag to separate multiple instances
during testing so that we can have two segregated testing environment on the same
Consul server.

Now let's copy from consul to tag for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r consul tag
cd ~/networknt/light-example-4j/discovery/api_b
cp -r consul tag
cd ~/networknt/light-example-4j/discovery/api_c
cp -r consul tag
cd ~/networknt/light-example-4j/discovery/api_d
cp -r consul tag
```

### API A

In order to register the service with a tag to indicate a separate environment. We
need to update server.yml with a environment tag.

```text
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

environment: test1

```  

Note that there is a newly added environment property with value test1. 



### API B

Let's update API B server.yml with environment tag.

```text
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

environment: test1
```

### API C

Let's update API C server.yml with environment tag.

```text
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

environment: test1

```

### API D

Let's update API D server.yml with environment tag.

```text
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

environment: test1

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
cd ~/networknt/light-example-4j/discovery/api_a/tag
mvn clean install -DskipTests
mvn exec:exec
```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/tag
mvn clean install -DskipTests
mvn exec:exec

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/tag
mvn clean install -DskipTests 
mvn exec:exec

```

API D


And start the first instance that listen to 7445 as default

```
cd ~/networknt/light-example-4j/discovery/api_d/tag
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

If pay attention you can find tag 'test1' for all services registered on Consul.


### Test four servers

```
curl -k https://localhost:7441/v1/data
```

And the result will be

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7445","API D: Message 2 from port 7445","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```
 
### Start another API D
 
Now let's start the second instance of API D. Before starting the serer, let's update
server.yml with port 7444 and with environment tag test2

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

environment: test2
```

And start the second instance

```
cd ~/networknt/light-example-4j/discovery/api_d/tag
mvn clean install -DskipTests 
mvn exec:exec

```

Take a look at the Consul UI, you can see the tag 'test2' for API D.

You can use the following command line to lookup API with test1 or test2 and you
will see one instance returns for each query.

```text
curl http://localhost:8500/v1/health/service/com.networknt.apid-1.0.0?passing&wait=600s&index=0&tag=test1
curl http://localhost:8500/v1/health/service/com.networknt.apid-1.0.0?passing&wait=600s&index=0&tag=test2
```

Can you guess what will be returned if you don't have tag query parameter? The
answer is all available instances will be returned. 


### Test Servers

As we cache the connection in our application, so even there is a new server added to
consul, it won't be used unless we create a new connection for each request. 

Let's issue the same command again. 

```
curl -k https://localhost:7441/v1/data
```

And the result should be the same.

```
["API C: Message 1","API C: Message 2","API D: Message 1 from port 7445","API D: Message 2 from port 7445","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

Shutdown the server with port 7445. Now when you call the same curl command, you will an
error although instance 7444 server is running.  

This is because that 7444 server has tag 'test2' and API B is looking up API D with tag
'test1'.

In summary, this step show you how to use environment tag to separate testing instances
for microservices. 

In the next step, Consul ACL will be enabled so that only services with token can register
and discover from it. 

