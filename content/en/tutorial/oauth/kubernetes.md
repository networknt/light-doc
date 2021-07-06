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

In the [start services][] section, we have introduced docker-comopse commands to start the light-oauth2 services with different databases. It is good for a dev and test environment, but not production. For production deployment, we are going to use Kubernetes or other Kubernetes implementations like Openshift. In this tutorial, we are going to walk through the deployment on a three nodes Kubernetes cluster. 


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

In most cases, we would recommend installing light-oauth2 into the VMs so that we can easily control how to form the cluster with the internal Hazelcast IMDG. In this tutorial, we will deploy all the services to the Kubernetes cluster and access these services from Kubernetes service names. There are numeric other ways to deploy the light-oauth2 to a Kubernetes cluster. We will explore them in other tutorials. 

### Docker Images

In this tutorial, we are going to use the docker images from https://hub.docker.com/u/networknt/dashboard/

1.5.19 is the latest tag at the moment when this tutorial is written. 

### Deployment Config

As light-oauth2 doesn't have any internal configuration files in the docker images, we need to prepare the externalized config files and also the deployment files for each service. 

We are going to use the light-config-test repo for this tutorial and the folder we are using is the following.

```
light-config-test/light-oauth2/kubernetes
```


[start services]: /tutorial/oauth/start/
