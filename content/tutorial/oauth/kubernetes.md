---
title: "Kubernetes Cluster"
date: 2018-07-04T22:45:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the [start services][] section, we have introduced docker-comopse commands to start the light-oauth2 services with different databases. It is good for dev and test environment but not production. For production deployment, we are going to use Kubernetes or other Kubernetes implementations like Openshift. In this tutorial, we are going to walk through the deployment on a three nodes Kubernetes cluster. 


### Introduction

Here is the Kubernetes cluster information. We are going to run the command line from sandbox which is the master node of the cluster. 

```
kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
node1           Ready     <none>    13d       v1.11.0
node2           Ready     <none>    13d       v1.11.0
node3           Ready     <none>    13d       v1.11.0
sandbox         Ready     master    13d       v1.11.0
```

In most of the cases, we would recommend to install light-oauth2 into the VMs so that we can easily control how to form the cluster. In this tutorial, we will deploy all the services to the first node using host network. After the deployment, we can access these services with the IP and default port numbers. There are numeric other ways to deploy the light-oauth2 to a Kubernetes cluster. You can deploy them to all three nodes dynamically and user service name to access each service. We will explore it in another tutorial. 

### Label Node

We want to deploy all light-oauth2 services with their default ports on the host network on node1 above. First we need to setup the nodeSelector label. 

```
kubectl label nodes node1 oauth2=dc1
```
The above give node1 a label oauth2=dc1 which means this is simulating data center 1 and we will deploy another data center in the subsequent tuturial to demo the federated light-oauth2 deployment across two different data centers. 

You can verify the result by running. 

```
kubectl get nodes --show-labels
```

### Docker Images

In this tutorail, we are going to use the docker images from https://hub.docker.com/u/networknt/dashboard/

1.5.18 is the latest tag at the moment when this tutorial is written. 

### Deployment Config

As light-oauth2 doesn't have any internal configuration files in the docker images, we need to prepare the externalized config files and also the deployment files for each service. 

We are going to use the light-config-test repo for this tutorial and the folder we are using is the following.

```

```


[start services]: /tutorial/oauth/start/
