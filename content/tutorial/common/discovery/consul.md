---
date: 2017-10-17T19:41:50-04:00
title: Consul service registry and discovery
---

Previous Step: [Multiple]({{< relref "/tutorial/common/discovery/multiple.md" >}})

In this step, we are going to use Consul for registry to enable cluster scaling and load balancing.

First, let's copy our current state from last step into the `consul` directory.

```bash
cp -r ~/networknt/discovery/api_a/multiple ~/networknt/discovery/api_a/consul
cp -r ~/networknt/discovery/api_b/multiple ~/networknt/discovery/api_b/consul
cp -r ~/networknt/discovery/api_c/multiple ~/networknt/discovery/api_c/consul
cp -r ~/networknt/discovery/api_d/multiple ~/networknt/discovery/api_d/consul
```


### API A

In order to switch from direct registry to consul registry, we just need to update
`service.yml` configuration to inject the consul implementation to the registry interface.

Let's make the following change in `service.yml`:

```yaml
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
```

Although in our case, there is no caller service for `API A`, we still need to register
it to consul by enable it in `server.yml`. 

Let's make the following changes in `server.yml`: 

```yaml
enableRegistry: true
```

### API B

Let's update `service.yml` to inject consul registry instead of direct registry used in
the previous step.

service.yml

```yaml
# Singleton service factory configuration/IoC injection
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
```

As `API B` will be called by `API A`, it needs to register itself to consul registry so
that `API A` can discover it through the same consul registry. To do that you only need
to enable server registry in config file and disable http connection.

server.yml

```yaml
enableRegistry: true
```

### API C

Although `API C` is not calling any other APIs, it needs to register itself to consul
so that `API A` can discovery it from the same consul registry.

`service.yml`

```yaml
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
```

`server.yml`

```yaml
enableRegistry: true
```

### API D

`service.yml`

```yaml
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
```

`server.yml`

```yaml
enableRegistry: true
```

### Start Consul

Here we are starting consul server in docker to serve as a centralized registry. To make it
simpler, the ACL in consul is disable by setting acl_default_policy=allow.

```bash
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

### Start four servers

Now let's start four terminals to start servers.  

**API A**

```bash
cd ~/networknt/discovery/api_a/consul
mvn clean install exec:exec
```

**API B**

```bash
cd ~/networknt/discovery/api_b/consul
mvn clean install exec:exec
```

**API C**

```bash
cd ~/networknt/discovery/api_c/consul
mvn clean install exec:exec
```

**API D**

And start the first instance that listen to 7444 as default

```bash
cd ~/networknt/discovery/api_d/consul
mvn clean install exec:exec
```

Now you can see the registered service from Consul UI.

```bash
http://localhost:8500/ui
```

### Test four servers

```bash
curl -k https://localhost:7441/v1/data
```

And the result will be

```bash
["API D: Message 1 from port 7445","API D: Message 2 from port 7445","API C: Message 1","API C: Message 2"]
```
 
### Start another API D
 
Now let's start the second instance of API D. Before starting the serer, let's update
`server.yml` with port 7444.

```yaml
httpsPort:  7444
```

And start the second instance

```bash
cd ~/networknt/discovery/api_d/consul
mvn clean install exec:exec
```

After you start the second instance of API D, you can go to the Web UI of consul to check if
there are two instances for the API D. 

### Test Servers

Let's issue the same command again. 

```bash
curl -k https://localhost:7441/v1/data
```

And the should be either of the following:

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API C: Message 1","API C: Message 2"]
["API D: Message 1 from port 7445","API D: Message 2 from port 7445","API C: Message 1","API C: Message 2"]
```

Shutdown the server with port 7445. Now when you call the same curl command, you will 
see 7444 server picks up the request.  

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API C: Message 1","API C: Message 2"]
```

In this step we have used Consul for API registration. When we call `API A `, all other APIs were discovered
from Consul to fulfill the request.
In the next step, we are going to start multiple instances that represent different environments
differentiated by tag.

Next Step: [Tags][]

[multiple]: /tutorial/common/discovery/multiple/
[Tags]: /tutorial/common/discovery/tag/
