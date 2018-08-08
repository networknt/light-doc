---
title: "Consul Production with TLS"
date: 2018-08-03T15:36:59-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous steps, we are using consul docker contain for service registry and discovery. It is ok for development, but for the production deployment, we have to setup Consul on at least three VMs as a cluster with TLS enabled. In this case, the connection from the service to the consul would be a little different. 

For the step to install Consul in production mode, please refer to [Consul Production Installation][].

To make things simpler, we are just going to copy the Kubernetes tutorial and change the internal config files. For production deployment, we need to externalize the config files to create Kubernetes ConfigMap or Secret. Or to leverage the light-config-server to manage all the externalized config files. 

### Create Consul TLS folder

Now let's copy from kubernetes to consul-tls for each API.
 
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

The consul.cert can be downloaded from any node of Consul cluster. Copy it into the src/main/resources/config/tls folder for each API and run the following import command. The default client.truststore password is "password".

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

If you are not familar with the [keytool][] command line or want to learn more about the [keystore and truststore][], please client the links for more details. Once import is done, we can remove the consul.cert from the tls folder. Repeat this for API A, B C and D. 


### Update ACL token

ACL token for the consul is configured in secret.yml file. Update all four files in api_a, api_b, api_c and api_d folders. You can get the agent ACL token from Consul admin. 

```
# Consul Token for service registry and discovery
consulToken: 3f5f1cef-2966-4964-73c5-7ebeb21ba337
```

### Switch to HTTPS for Consul Registry

Update the service.yml file to switch Consul connection to HTTPS from HTTP. Please update four files from api_a, api_b, api_c and api_d folders. We also need to update the consul IP address in the same file. As we don't have DNS set up yet, we can just use the IP address now. In order to simulate the dynamic nature of connections. We are going to setup api_a and api_d to connect to 198.55.49.188, api_b to connect to 198.55.49.187 and api_c to connect to 198.55.49.186. 

Here is the section in service.yml for api_a and api_d

```
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: https
    - java.lang.String: 198.55.49.188
    - int: 8500

```

api_b

```
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: https
    - java.lang.String: 198.55.49.187
    - int: 8500
```

api_c

```
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl:
    - java.lang.String: https
    - java.lang.String: 198.55.49.186
    - int: 8500
```

### Build Docker Images

As we just updated the internal configuration, so the deployment process would be the same like [Kubernetes][] step after rebuilding the docker images. 

Now let's build and publish the image for api_d locally without publish it to the docker hub. 

```
cd ~/networknt/light-example-4j/discovery/api_d/consul-tls
mvn clean install
./build.sh 1.5.18-ct
```

We are using 1.5.18-ct to ensure that the image won't replace the normal 1.5.18 image. 

Let's do the above steps for api_c, api_b and api_a. 

### Deploy Services

Now, we need to update the deployment file to specify the image with tag. 

For example, the apid-deployment.yaml will be using image like below. 


```
image: networknt/com.networknt.apid-1.0.0:1.5.18-ct
```

If you follow the tutorial and completed the step [Kubernetes][], you will have the docker image pulled to your Kubernetes master node. In this case, you want to force the Kubernetes to pull the new image. 

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
        image: networknt/com.networknt.apid-1.0.0:1.5.18-ct
        imagePullPolicy: Always
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
```

Please note that imagePullPolicy is set as Always. Let's update api_a, api_b and api_c for the deployment script. 

Login to the kubernetes master node and checkout the latest light-example-4j. 

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




[Consul Production Installation]: /tool/consul/cluster-install/
[keytool]: /tool/keytool/
[keystore and truststore]: /tutorial/security/keystore-truststore/
[Kubernetes]: /tutorial/common/discovery/kubernetes/

