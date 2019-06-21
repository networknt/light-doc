---
title: "Consul Production with TLS"
date: 2018-08-03T15:36:59-04:00
description: "Service discovery with a production ready Consul cluster with TLS enabled from Kubernetes cluster."
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous steps, we are using consul docker container for service registry and discovery. It is ok for development, but for the production deployment, we have to setup Consul on at least three VMs as a cluster with TLS and ACL enabled. In this case, the connection from the service to the consul would be a little different. 

For the step to install Consul in production mode, please refer to [Consul Production Installation][].

To make things simpler, we are just going to copy the Kubernetes tutorial and change the internal config files. For production deployment, we need to externalize the config files to create Kubernetes ConfigMap or Secret. Or to leverage the light-config-server to manage all the externalized config files. To simplify this tutorial, we are just going to use the internal config files for now. To learn how to externalize config files, please refer to [External Config][] tutorial. 

### Create Consul TLS folder

Now let's copy from Kubernetes to consul-tls for each API.
 
```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r kubernetes consul-tls
cd ~/networknt/light-example-4j/discovery/api_b
cp -r kubernetes consul-tls
cd ~/networknt/light-example-4j/discovery/api_c
cp -r kubernetes consul-tls
cd ~/networknt/light-example-4j/discovery/api_d
cp -r kubernetes consul-tls
```

### Import Certificate

The certificate needs to be imported to the client.truststore for each API in order to connect to the consul cluster. 

The consul.cert can be downloaded from any node of Consul cluster. Copy it into the src/main/resources/config/tls folder for each API and run the following import command. The default client.truststore password is "password". As we are using a self-signed cert with a CA cert, we need to import both the consul.cert and ca.cert. 

```
cd ~/networknt/light-example-4j/discovery/api_d/consul-tls/src/main/resources/config/tls
scp consul1:/etc/consul.d/ssl/consul.cert .
keytool -import -trustcacerts -alias consul -file consul.cert -keystore client.truststore
rm consul.cert

cd ~/networknt/light-example-4j/discovery/api_d/consul-tls/src/main/resources/config/tls
scp consul1:/etc/consul.d/ssl/ca.cert .
keytool -import -trustcacerts -alias consul-ca -file ca.cert -keystore client.truststore
rm ca.cert

```

If you are not familiar with the [keytool][] command line or want to learn more about the [keystore and truststore][], please click the links for more details. Once the certificates are imported, we can remove the consul.cert and ca.cert from the tls folder. Repeat this for API A, B and C. 


### Update ACL token

ACL token for the consul is configured in the consul.yml file. Update all four files in api_a, api_b, api_c and api_d consul-tls folders. You can get the agent ACL token from Consul admin. 

Here is the one from our consul cluster. 

```
# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

### Switch to HTTPS for Consul Registry

Update the consul.yml file to switch Consul connection to HTTPS from HTTP. Please update four files from consul-tls in api_a, api_b, api_c and api_d folders. We also need to update the consul IP address in the same file. As we don't have DNS set up yet, we can use the IP address now. To simulate the dynamic nature of connections, we are going to setup api_a and api_d to connect to 198.55.49.188, api_b to connect to 198.55.49.187 and api_c to connect to 198.55.49.186. 

Here is the url in consul.yml for api_a and api_d

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
```

Here is the url and token in consul.yml for api_b

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.187:8500
# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

Here is the url and token in consul.yml for api_c

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.186:8500
# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

### Build Docker Images

As we just updated the internal configuration, the deployment process would be the same as [Kubernetes][] step after rebuilding the docker images. 

Now let's build and publish the image for api_d. 

```
cd ~/networknt/light-example-4j/discovery/api_d/consul-tls
mvn clean install
./build.sh 2.0.2-ct
```

We are using 2.0.2-ct to ensure that the image won't replace the normal 2.0.2 image. 

Let's do the above steps for api_c, api_b and api_a. 

### Deploy Services

Now, we need to update the deployment file to specify the image with tag. 

For example, the apid-deployment.yaml will be using the image like below. 


```
image: networknt/com.networknt.ad-1.0.0:2.0.2-ct
```

If you followed the tutorial and completed the step [Kubernetes][], you will have the docker image pulled to your Kubernetes master node. In this case, you want to force the Kubernetes to pull the new image. 

Here is the updated apid-deployment.yaml

```
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
        image: networknt/com.networknt.ad-1.0.0:2.0.2-ct
        imagePullPolicy: Always
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
```

Please note that imagePullPolicy is set as Always. Let's update api_a, api_b, and api_c for the deployment script. 

Login to the Kubernetes master node and check out the latest light-example-4j. 

To start six replicas per service, run the following commands.

```
cd ~/networknt/light-example-4j
git pull origin develop
cd ~/networknt/light-example-4j/discovery/api_d/consul-tls
kubectl create -f apid-deployment.yaml
cd ~/networknt/light-example-4j/discovery/api_c/consul-tls
kubectl create -f apic-deployment.yaml
cd ~/networknt/light-example-4j/discovery/api_b/consul-tls
kubectl create -f apib-deployment.yaml
cd ~/networknt/light-example-4j/discovery/api_a/consul-tls
kubectl create -f apia-deployment.yaml
```

### Test Services

Now let's go to the consul UI to find an instance of api_a. 

```
https://198.55.49.188:8500/ui/dc1/services/com.networknt.aa-1.0.0
```

Click the instance and find out the IP and port number. Now, let's construct a curl command to test the service chain. 

```
curl -k https://38.113.162.53:2406/v1/data
```

The result should be something like the following. 

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

### Summary

In this tutorial, we have walked through the configurations of the four services to work with a Consul cluster that supports TLS and ACL. All config files are updated internally and packaged into the Docker container. In the next step, we are going to use the externalized config files in [External Config][] tutorial.


[Consul Production Installation]: /tool/consul/cluster-install/
[keytool]: /tool/keytool/
[keystore and truststore]: /tutorial/security/keystore-truststore/
[Kubernetes]: /tutorial/common/discovery/kubernetes/
[External Config]: /tutorial/common/discovery/external-config/
