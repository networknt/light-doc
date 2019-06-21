---
date: 2017-10-19T19:57:17-04:00
title: Kubernetes
description: "Deploying services to Kubernetes and using Consul for service registry and discovery"
reviewed: true
weight: 90
sections_weight: 90
draft: false
toc: true
reviewed: true
---

In the previous step, we have explored docker compose with multiple services, and it works very well on a user's laptop or desktop. In this section, we are going to explore how to deploy these services to a Kubernetes cluster with multiple instances of each and see how they can discover each other. 

First, let's create a Kubernetes cluster with one master and three worker nodes. For the steps to install Kubernetes, please follow [Kubernetes][] in the tool section. If you do not have access to multiple VMs to create a real cluster, you can use [Minikube][] for this tutorial, but I have not tried it myself. 

This step is for production use; however, to simplify the discovery, I have not enabled JWT token verification. All communication is in HTTPS and HTTP 2.0 though.

Kubernetes provides many components to abstract different layers and to simplify the service interactions; however, it might not be the fastest way for high-speed services. In most of the conditions, a Layer 7 proxy is used to load balance the traffic between nodes and pods. For normal usage, this is OK, but if you want to scale, the proxy hop significantly hinder your throughput. If you are doing real microservices which means you have dozens of services involved in most requests coming from outside, you want to bypass these proxies and establish the connection directly and cache it for a while. It is one of the principal reasons we have used client-side discovery instead of server-side discovery. Another reason is to support environment segregation with tags which we have shown in the [tag][] section.    

### Environment

Here is the information about our Kubernetes cluster. 

```
steve@sandbox:~$ kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
38-113-162-51   Ready     <none>    4d        v1.11.0
38-113-162-52   Ready     <none>    4d        v1.11.0
38-113-162-53   Ready     <none>    4d        v1.11.0
sandbox         Ready     master    4d        v1.11.0
```

### Install Consul

We are going to install Consul on sandbox just for testing purpose. Usually, you should not install any container in the sandbox as it is Kubernetes master. We are just using it to simulate that Consul is installed in a data center with a static IP address. 

```
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

To verify that Consul is up and running from the Web UI, go to your browser and paste the following link. 

```
http://sandbox:8500/ui/#/dc1/services
```

### Kubernetes folder

Now let's copy from consuldocker to kubernetes for each API.
 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r consuldocker kubernetes
cd ~/networknt/light-example-4j/discovery/api_b
cp -r consuldocker kubernetes
cd ~/networknt/light-example-4j/discovery/api_c
cp -r consuldocker kubernetes
cd ~/networknt/light-example-4j/discovery/api_d
cp -r consuldocker kubernetes
```

### API D

In order to register on Consul, our server must know the external IP address for the node and a port number that is bound to the IP address. In some scenarios, we need to config server.yml to set up an environment tag to separate different environment in the same Kubernetes cluster for tests.

As different pods can be deployed on one Kubernetes node, and they might want to bind to the same port. In order to avoid conflict, we are going to use dynamic port allocation and let the server to select a port that it can be bound. 

Let's change the server.yml to the following. 

```yaml

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 7004

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 7444

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Keystore password
keystorePass: password

# Private key password
keyPass: password

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Truststore password
truststorePass: password

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.ad-1.0.0

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

# Build Number
# Allows teams to audit the value and set it according to their release management processes
buildNumber: 1.0.0
``` 

As you can see, enableRegistry is true and dynamicPort is true. A range of ports has been specified by providing minPort and maxPort. 

Now let's change the consul.yml to the following. 

```
# Consul URL for accessing APIs
consulUrl: http://consul:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

Please note that we are using consul as the host name to consul server on the sandbox. Also we have change health check to tcpCheck so that consul server will try to ping all services to sure they are up and running. 

Now let's create a Docker image and upload it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_d/kubernetes
mvn clean install
./build.sh 2.0.2
```

Next, let's create a Kubernetes Deployment file. This should be located in a separate repo for deployment and that repo should be accessed from sandbox for deployment. However, for now, let's just put it into api_d/kubernetes folder for reference only.

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: apid-deployment
  labels:
    app: apid
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apid
  template:
    metadata:
      labels:
        app: apid
    spec:
      hostNetwork: true
      containers:
      - name: apid
        image: networknt/com.networknt.ad-1.0.0
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP

```

Login to the sandbox which is the master of the Kubernetes cluster and call kubectl to create the deployment.  

```
kubectl create -f apid-deployment.yaml
```

And check how many pods are created with.

```
kubectl get pods | grep apid
```

The result should be something like the following. 

```
apid-deployment-cd57b9b45-5x4qd   1/1       Running   0          21s
apid-deployment-cd57b9b45-bh2xw   1/1       Running   0          21s
apid-deployment-cd57b9b45-btps8   1/1       Running   0          21s
apid-deployment-cd57b9b45-h6lw2   1/1       Running   0          21s
apid-deployment-cd57b9b45-mrjv4   1/1       Running   0          21s
apid-deployment-cd57b9b45-r9ws9   1/1       Running   0          21s

```

Now if you go to Consul Web UI, you should see api_d is registered with 6 instances. Note that in service.yml the Consul server host is consul and you must ensure that you have an entry on three workers /etc/hosts to point to consul.

If you want to check the log for one of the instances, you can do that with the following.

```
kubectl logs apid-deployment-cd57b9b45-5x4qd
``` 

From the log, you can see which host and port this particular instance is bound to. To test it, replace the following ip and port.

```
curl -k https://ip:port/v1/data
```

You should have something like this.

```json
["API D: Message 1 from port 7444","API D: Message 2 from port 7444"]
```

### API C

Let's change the server.yml to the following. 

```yaml
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 7003

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 7443

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Keystore password
keystorePass: password

# Private key password
keyPass: password

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Truststore password
truststorePass: password

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.ac-1.0.0

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

# Build Number
# Allows teams to audit the value and set it according to their release management processes
buildNumber: 1.0.0

``` 

As you can see, enableRegistry is true and dynamicPort is true. A range of ports has been specified by providing minPort and maxPort. 

Now let's change the consul.yml to the following. 

```
# Consul URL for accessing APIs
consulUrl: http://consul:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

Please note that we are using consul as the host name to consul server on the sandbox. Also we have change health check to httpCheck so that consul server will try to ping all services to sure they are up and running. 

Now let's create a Docker image and upload it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_c/kubernetes
mvn clean install
./build.sh 2.0.2
```

Next, let's create a Kubernetes Deployment file. This should be in a separate repo for deployment and that repo should be accessed from sandbox for deployment. However, for now, let's just put it into api_c/kubernetes folder for reference only.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apic-deployment
  labels:
    app: apic
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apic
  template:
    metadata:
      labels:
        app: apic
    spec:
      hostNetwork: true
      containers:
      - name: apic
        image: networknt/com.networknt.ac-1.0.0
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP

```

In the above deployment file, you can see that we pass the status.hostIP to the container so that the light-4j server can register itself to Consul using this IP address. 

Login to the sandbox which is the master of the Kubernetes cluster and call kubectl to create the deployment.  

```
kubectl create -f apic-deployment.yaml
```

And check how many pods are created with.

```
kubectl get pods | grep apic
```

The result should be something like the following. 

```
apic-deployment-55b97c5b6d-4ccb2   1/1       Running   0          9s
apic-deployment-55b97c5b6d-c5w6m   1/1       Running   0          9s
apic-deployment-55b97c5b6d-kthxl   1/1       Running   0          9s
apic-deployment-55b97c5b6d-lsmkf   1/1       Running   0          9s
apic-deployment-55b97c5b6d-s8njj   1/1       Running   0          9s
apic-deployment-55b97c5b6d-v2hct   1/1       Running   0          9s

```

Now if you go to Consul Web UI, you should see api_c is registered with 6 instances. Note that in service.yml the Consul server host is consul and you must ensure that you have an entry on the three workers /etc/hosts to point to consul.

If you want to check the log for one of the instances, you can do that with the following.

```
kubectl logs apic-deployment-55b97c5b6d-4ccb2
``` 

From the log, you can see which host and port this particular instance is bound to. To test it, replace the following ip and port.

```
curl -k https://ip:port/v1/data
```

You should have result like the following. 

```json
["API C: Message 1","API C: Message 2"]
```

### API B

Let's change the server.yml to the following. 

```yaml

# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 7002

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 7442

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Keystore password
keystorePass: password

# Private key password
keyPass: password

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Truststore password
truststorePass: password

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.ab-1.0.0

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

# Build Number
# Allows teams to audit the value and set it according to their release management processes
buildNumber: 1.0.0

``` 

As you can see, enableRegistry is true and dynamicPort is true. A range of ports has been specified by providing minPort and maxPort. 

Now let's change the consul.yml to the following. 

```
# Consul URL for accessing APIs
consulUrl: http://consul:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

Please note that we are using consul as the host name to consul server on the sandbox. Also we have change health check to httpCheck so that consul server will try to ping all services to sure they are up and running. 


Now let's create a Docker image and upload it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_b/kubernetes
mvn clean install
./build.sh 2.0.2
```

Next, let's create a Kubernetes Deployment file. This should be in a separate repo for deployment and that repo should be accessed from sandbox for deployment. However, for now, let's just put it into api_b/kubernetes folder for reference only.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apib-deployment
  labels:
    app: apib
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apib
  template:
    metadata:
      labels:
        app: apib
    spec:
      hostNetwork: true
      containers:
      - name: apib
        image: networknt/com.networknt.ab-1.0.0
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP

```

In the above deployment file, you can see that we pass the status.hostIP to the container so that the light-4j server can register itself to Consul using this IP address. 

Login to the sandbox which is the master of the Kubernetes cluster and call kubectl to create the deployment.  

```
kubectl create -f apib-deployment.yaml
```

And check how many pods are created with.

```
kubectl get pods | grep apib
```

The result should be something like the following. 

```
apib-deployment-55b46dd8f9-4wjpp   1/1       Running   0          10s
apib-deployment-55b46dd8f9-65g2f   1/1       Running   0          10s
apib-deployment-55b46dd8f9-gf5q2   1/1       Running   0          10s
apib-deployment-55b46dd8f9-kqtqx   1/1       Running   0          10s
apib-deployment-55b46dd8f9-p4trm   1/1       Running   0          10s
apib-deployment-55b46dd8f9-tn54m   1/1       Running   0          10s

```

Now if you go to Consul Web UI, you should see api_b is registered with 6 instances. Note that in service.yml the Consul server host is consul and you must ensure that you have an entry on the three workers /etc/hosts to point to consul.

If you want to check the log for one of the instances, you can do that with the following.

```
kubectl logs apib-deployment-55b46dd8f9-4wjpp
``` 

From the log, you can see which host and port this particular instance is bound to. To test it, replace the following ip and port.

```
curl -k https://ip:port/v1/data
```

And the result should be something like the following.

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2"]
```

As you can see that API B is discovered an instance of API D and called it. 


### API A


Let's change the server.yml to the following. 

```yaml
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort: 7001

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort: 7441

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Keystore password
keystorePass: password

# Private key password
keyPass: password

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Truststore password
truststorePass: password

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.aa-1.0.0

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

# Build Number
# Allows teams to audit the value and set it according to their release management processes
buildNumber: 1.0.0

``` 

As you can see, enableRegistry is true and dynamicPort is true. A range of ports has been specified by providing minPort and maxPort. 

Now let's change the consul.yml to the following. 

```
# Consul URL for accessing APIs
consulUrl: http://consul:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

Please note that we are using consul as the host name to consul server on the sandbox. Also we have change health check to tcpCheck so that consul server will try to ping all services to sure they are up and running. 

Now let's create a Docker image and upload it to docker hub.

```
cd ~/networknt/light-example-4j/discovery/api_a/kubernetes
mvn clean install
./build.sh 2.0.2
```

Next, let's create a Kubernetes Deployment file. This should be in a separate repo for deployment and that repo should be accessed from sandbox for deployment. However, for now, let's just put it into api_a/kubernetes folder for reference only.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apia-deployment
  labels:
    app: apia
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apia
  template:
    metadata:
      labels:
        app: apia
    spec:
      hostNetwork: true
      containers:
      - name: apia
        image: networknt/com.networknt.aa-1.0.0
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP

```

In the above deployment file, you can see that we pass the status.hostIP to the container so that the light-4j server can register itself to Consul using this IP address. 

Login to the sandbox which is the master of the Kubernetes cluster and call kubectl to create the deployment.  

```
kubectl create -f apia-deployment.yaml
```

And check how many pods are created with.

```
kubectl get pods | grep apia
```

The result should be something like the following. 

```
apia-deployment-d57454d6-jq4qq   1/1       Running   0          5m
apia-deployment-d57454d6-k9z54   1/1       Running   0          5m
apia-deployment-d57454d6-pn92r   1/1       Running   0          5m
apia-deployment-d57454d6-r9k6l   1/1       Running   0          5m
apia-deployment-d57454d6-tphms   1/1       Running   0          5m
apia-deployment-d57454d6-x9mgq   1/1       Running   0          5m


```

Now if you go to Consul Web UI, you should see api_a is registered with 6 instances. Note that in service.yml the Consul server host is consul and you must ensure that you have an entry on three workers /etc/hosts to point to consul.

If you want to check the log for one of the instances, you can do that with the following.

```
kubectl logs apia-deployment-d57454d6-jq4qq
``` 

From the log, you can see which host and port this particular instance is bound to. To test it, replace the following ip and port.

```
curl -k https://ip:port/v1/data
```

And the result should be something like the following.

```json
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

As you can see, all four services are invoked to fulfill the request. 

### Scale Down

In above steps, we have created 6 replicas for each service and deployed them on three nodes to demo the ability for the light-4j framework to start multiple instances on the same host with dynamic port allocation within a given range. 

Now let's scale down to one instance of each service and see if our application can still working by doing the client side discovery and failover. 

```
kubectl scale --replicas=1 -f apid-deployment.yaml
```

And issue the same command to apia will still get result.  

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Now try to scale down all service to only one instance and test it again. Be sure to find the only instance of apia to call the host:port combination. 

### Scale Up

Let's now scale up to 3 replicas for each service. 

```
kubectl scale --replicas=3 -f apid-deployment.yaml
kubectl scale --replicas=3 -f apic-deployment.yaml
kubectl scale --replicas=3 -f apib-deployment.yaml
kubectl scale --replicas=3 -f apia-deployment.yaml

```

Issue kubectl get pods will get the following.

```
NAME                               READY     STATUS    RESTARTS   AGE
apia-deployment-d57454d6-9m9lg     1/1       Running   0          16s
apia-deployment-d57454d6-crvgv     1/1       Running   0          16s
apia-deployment-d57454d6-qkxjf     1/1       Running   0          1h
apib-deployment-55b46dd8f9-4wjpp   1/1       Running   0          20m
apib-deployment-55b46dd8f9-krrj5   1/1       Running   0          17s
apib-deployment-55b46dd8f9-vf6c7   1/1       Running   0          17s
apic-deployment-55b97c5b6d-kthxl   1/1       Running   0          28m
apic-deployment-55b97c5b6d-txfg5   1/1       Running   0          19s
apic-deployment-55b97c5b6d-zgmx7   1/1       Running   0          19s
apid-deployment-cd57b9b45-4b8qb    1/1       Running   0          20s
apid-deployment-cd57b9b45-btps8    1/1       Running   0          46m
apid-deployment-cd57b9b45-rnnrn    1/1       Running   0          20s

``` 

This tutorial shows how to deploy services to Kubernetes cluster and how to look up these services from Consul. For service to service communication, client module is used to perform client-side service discovery. In the situation that client module cannot be used, for example, the client is not built on Java 8 or not even in Java, light-router can be used to assist client-side service discovery.  Depending on the ownership of light-router, it can handler the JWT token renewal as well. For more details, please refer to [router][] tutorial.  

[Kubernetes]: /tool/kubernetes/
[Minikube]: /tool/minikube/
[tag]: /tutorial/common/discovery/tag/
[router]: /tutorial/common/discovery/router/
