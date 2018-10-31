---
title: "Light-oauth2 on Kubernetes"
date: 2018-10-31T10:45:01-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This section describes the steps to deploy light-oauth2 microservices to a Kubernetes (public cloud)  or Openshift (on-premise)  cluster as services.  

If you want to take a look at the detail configuration, please check out https://github.com/networknt/light-config-test/tree/master/light-oauth2/openshift

Several things that need to be addressed in the deployment process. 

### Hazelcast

As we are using Hazelcast IMDG to sync between services, we need to make sure that it is working properly in Kubernetes cluster. 

For each service, you need to have an exteralized hazelcast.xml in the config folder. You must have the following section although the Hazelcast document says it is not necessary. 

```
    <properties>
        <property name="hazelcast.discovery.enabled">true</property>
    </properties>
```

Multi-cast normally won't be enabled in a Kubernetes namespace or Openshift project, so we have to use 
Kubernetes plugin for discovery. You need to disable all other options and add this section. 

```
<discovery-strategies>
                <!-- activate the Kubernetes plugin -->
                <discovery-strategy enabled="true" class="com.hazelcast.kubernetes.HazelcastKubernetesDiscoveryStrategy">
                    <properties>
                        <!-- configure discovery service API lookup -->
                        <!-- <property name="service-name">MY-SERVICE-NAME</property> -->
                        <!-- as we have multiple services with different names, skip this -->
                        <!-- https://github.com/hazelcast/hazelcast-kubernetes#rest-api-request -->
                        <property name="service-label-name">light-oauth2-cluster01</property>
                        <property name="service-label-value">true</property>
                        <property name="namespace">dit1</property>
                    </properties>
                </discovery-strategy>                
            </discovery-strategies>
```

You can update your own service-label-name, service-lable-value and namespace. They must match to your deployment configuration. 


In your service configuration, you need to specify the label. Here is an example oauth2-client-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: ../../bin/kompose --provider openshift --file ../docker-compose-oauth.yml
      convert
    kompose.version: ""
  creationTimestamp: null
  labels:
    io.kompose.service: oauth2-client
    light-oauth2-cluster01: true
  name: oauth2-client
spec:
  ports:
  - name: "6884"
    port: 6884
    targetPort: 6884
  selector:
    io.kompose.service: oauth2-client
status:
  loadBalancer: {}
```

Please note that the light-oauth2-cluster01 is set to true which match the discovery properties in hazelcast.xml

### Permission

You might encouter permission issue for Hazelcast service discovery. 

For default Openshift environment, the default user doesn't have permission to access Kubernetes API. If you see the following error, please follow the steps to resolve it. 

```
SEVERE: [10.101.111.226]:5701 [uat] [3.9.2] Failure executing: GET at: https://kubernetes.default.svc/api/v1/namespaces/uat1/endpoints?labelSelector=light-oauth2-cluster01%3Dtrue . Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. User "system:serviceaccount:uat1:default" cannot list endpoints in the namespace "uat1": User "system:serviceaccount:uat1:default" cannot list endpoints in project "uat1".
io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: GET at: https://kubernetes.default.svc/api/v1/namespaces/uat1/endpoints?labelSelector=light-oauth2-cluster01%3Dtrue . Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. User "system:serviceaccount:uat1:default" cannot list endpoints in the namespace "uat1": User "system:serviceaccount:uat1:default" cannot list endpoints in project "uat1".
 
```

You can add project read-only access to the ‘default’ serviceaccount – from an Openshift project admin role. 

```
oc policy add-role-to-user view -n uat1 -z default
```
 
Now you’ll be able to access the URL with your ‘default’ SA token.
 
``` 
curl -k -H "Authorization: Bearer <default SA token>" https://kubernetes.default.svc/api/v1/namespaces/uat1/endpoints
```


