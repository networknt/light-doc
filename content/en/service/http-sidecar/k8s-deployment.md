---
title: "K8s Deployment"
date: 2022-02-21T22:25:17-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When working on complicated Kubernetes deployment, we need to choose a deployment engine. There are two types: Templating Engine and Overlay Engine. 

Helm is the best implementation of Templating Engine, and Kustomize is the best implementation of the Overlay Engine. 

Here are the pros and cons of each tool. 

| Features                    | Kustomize  | Helm           |
| --------------------------- | ---------: | -------------: |
| Native K8s integration      | Yes        | No             |
| Overlays                    | Yes        | No             |
| Visibility and transparentcy| Strong     | Weak           |
| Packaging                   | No         | Yes            |
| Version control, rollbacks  | No         | Yes            |
| Templating                  | No         | Yes            |
| Target user                 | DevOps     | Developer      |


The most important is the target user with all the pros and cons above. To use Kustomize, you need to deeply understand Kubernetes YAML specifications to patch them at certain places. It is a very good tool for someone who is an expert on K8s. Most developers will be more comfortable with a templating engine. With a proper template, they only need to create a values.yml to define several property values for each environment. 

For a big organization with many teams working on different applications with a shared DevOps team, it is good to choose a templating engine. However, the Helm chart has its problem extending to something not defined in the original template. The Helm template will be hard to read if teams add their customization blocks conditionally. Here is an example.

```
rollingUpdate:
  maxSurge: 1
  {{ if gt .Values.replicaCount 2.0}}
  maxUnavailable: 0
  {{ else }}
  maxUnavailable: 1
  {{ end }}

```

And sometimes, you need to typecast values in your YAML config. 


```
...
ports:
- containerPort: !!int {{ .Values.containers.app.port }}
...
```

The reason we have the above complicated template is due to the Helm Chart being designed two-dimensional. The DevOps team needs to develop a template that will work for all environments, and the project team needs to come up with all the values to ensure the template is filled with values before deployment. 

Since we have the config server already, we can use it to manage the templates and values in multiple-dimensional. 

We can define the template for http-sidecar, and we can also make slight changes at the project level, project version level, service level, service version level, environment level when necessary.

We can also define the global values at http-sidecar level, project level, project version level, service level, service version level, and environment level. For a real service deployment on a target environment, maybe there would only be two or three values that need to be overwritten from the global default values. 

Here is an example of Kubernetes Deployment. 

deployment.yml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${deployment.name}
  labels:
    app: ${deployment.name}
spec:
  replicas: ${deployment.replicas:1}
  selector:
    matchLabels:
      app: ${deployment.name}
  template:
    metadata:
      labels:
        app: ${deployment.name}
    spec:
      containers:
        - name: ${deployment.name}-backend
          image: ${deployment.backend.name:networknt/com.networknt.petstore-3.0.1}:{deployment.backend.tag:1.0.0}
          imagePullPolicy: Always
          ports:
            - containerPort: {deployment.backend.port:8090}
        - name: ${deployment.name}-sidecar
          image: ${deployment.sidecar.name:networknt/com.networknt.http-sidecar}:{deployment.sidecar.tag:2.0.32}
          imagePullPolicy: Always
          ports:
            - name: sidecar-http
              containerPort: {deployment.sidecar.httpPort:8080}
            - name: sidecar-https
              containerPort: {deployment.sidecar.httpsPort:8443}
```


service level values.yml

```
deployment.name: petstore
```

environment level values.yml

```
deployment.replicas: 2
deployment.backend.tag: 1.2.0
```


