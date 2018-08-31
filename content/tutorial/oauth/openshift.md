---
title: "OpenShift Deployment"
date: 2018-08-29T14:05:25-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The configuration files and deployment script can be found at light-config-test repo. 

```
light-config-test/light-oauth2/openshift
```

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



