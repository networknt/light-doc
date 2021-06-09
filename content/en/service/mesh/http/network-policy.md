---
title: "Network Policy"
date: 2021-06-09T14:29:59-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When http-sidecar is used, all the traffic to and from the pod should go through the sidecar for API invocation and invoke other APIs. Here is an example network policy on the Kubernetes cluster. 

networkpolicy.yaml
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: proxy-allow
  labels:
    app: api
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector: {}
    ports:
    - protocol: TCP
      port: 8080

  egress:
  - to:
    ports:
    - protocol: TCP
      port: 8438
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 8443
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  policyTypes:
  - Ingress
  - Egress              
```

For ingress rules, it allows both North-South traffic from the Internet with `ingress-nginx`, for example, and east-west traffic between APIs within the same Kubernetes cluster regardless of the namespace on port 8080. 

For egress rules, we block all traffic by default and only allow certain ports to go outside. The following ports are examples for most organizations with the on-prem cluster. 

Port 8438 is used for the sidecar to connect to the Control Pane for the service registry and discovery.
Port 443 is used to interact with the OAuth 2.0 provider to get the JWK or JWT token.
Port 8443 is an optional port used to proxy the request from the Kubernetes cluster to the OAuth 2.0 provider if a proxy is used.
Port 53 is used to resolve the DNS name to IP address for OAuth 2.0 provider. Both TCP and UDP are opened here.

With the above network policy, we only allow Egress for certain ports. For some APIs, they might need to access databases or Kafka clusters etc. The solution is the define its API-specific network policy to add these ports to the Egress. It means the API needs to have an overlay with a customized network policy YAML file. During the deployment, the API-specific network policy will be merged with the proxy network policy, and the additional ports will be added to the Egress. 
