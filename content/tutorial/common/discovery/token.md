---
date: 2017-10-17T21:03:55-04:00
title: Access consul with acl_token for security
---

In previous steps, we have set up the Consul with acl_default_policy=allow so that
all operations to the Consul server is allow. This should be only used internal forr
testing. For official environment, the acl_default_policy=deny and all operation to
the Consul server must provide an acl_token in the header.


Now let's copy from tag to token for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r tag token
cd ~/networknt/light-example-4j/discovery/api_b
cp -r tag token
cd ~/networknt/light-example-4j/discovery/api_c
cp -r tag token
cd ~/networknt/light-example-4j/discovery/api_d
cp -r tag token
```

If you consul server is still running, please stop it and restart it with the following

```text
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"deny","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

Now let's create agent token

```text
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "Name": "Agent Token",
  "Type": "client",
  "Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"read\" }"
}' http://127.0.0.1:8500/v1/acl/create
```

And we get the token like this.

```text
{"ID":"67ccac85-36a3-e912-d0b3-bce1194119a0"}
```

Introduce the token through an API.

```text
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "Token": "67ccac85-36a3-e912-d0b3-bce1194119a0"
}' http://127.0.0.1:8500/v1/agent/token/acl_agent_token
```


Give anonymous read permission

```text
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "ID": "anonymous",
  "Type": "client",
  "Rules": "node \"\" { policy = \"read\" }"
}' http://127.0.0.1:8500/v1/acl/update

```

It should return something like this.

```json
{"ID":"anonymous"}
```

The next step we are going to update secret.yml to add consul-token in order to access
the Consul for service registry and discovery.

### API A

Update secret.yml to add consulToken: the_one_ring

```yaml
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
consulToken: the_one_ring

# EmailSender password
emailPassword: change-to-real-password

```

### API B

Update secret.yml to add consulToken: the_one_ring

```yaml
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
consulToken: the_one_ring

# EmailSender password
emailPassword: change-to-real-password

```

### API C

Update secret.yml to add consulToken: the_one_ring

```yaml
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
consulToken: the_one_ring

# EmailSender password
emailPassword: change-to-real-password

```

### API D

Update secret.yml to add consulToken: the_one_ring

```yaml
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
consulToken: the_one_ring

# EmailSender password
emailPassword: change-to-real-password

```

### Start four servers

Now let's start four terminals to start servers.  

API A

```
cd ~/networknt/light-example-4j/discovery/api_a/token
mvn clean install -DskipTests
mvn exec:exec
```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/token
mvn clean install -DskipTests
mvn exec:exec

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/token
mvn clean install -DskipTests 
mvn exec:exec

```

API D


And start the first instance that listen to 7444 as default

```
cd ~/networknt/light-example-4j/discovery/api_d/token
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

This is not working so far and need to work it out.


In this step, we tried to secure consul access from services so that we can be sure that
the services registered on the consul are the valid ones. In the next step, we are going
to start services with [Docker][] containers.

[Docker]: /tutorial/common/discovery/docker/
