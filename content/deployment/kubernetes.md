---
title: "Deploy to Kubernetes"
date: 2018-04-30T22:02:40-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "deployment"
    weight: 70
weight: 70
aliases: []
toc: false
draft: false
reviewed: true
---

Although you can deploy the microservices built on top of the light platform to a data center with `java -jar` or with Docker, the ultimate deployment platform would be a Kubernetes cluster. 

Before deploying a service to a Kubernetes cluster, you need to create secrets with all the externalized configuration files. These files usually are copied to config folder in each project in [light-config-test][] or [light-config-prod][]. For detailed directory structure, please refer to [petstore][]. 

### Secrets

To create secrets, make sure you have all the configuration files in the config folder. Then run the following create_secrets.sh

```
kubectl create secret generic petstore-secret --from-file=config
```

The above script creates a generic secret object with all the config files in the config folder. Each file name will be the key, and the base64 encoded content will be the value. 

To list all the secrets: 

```
kubectl get secrets
```

To view the content of a specific secret:

```
kubectl get secret petstore-secret -o yaml
```

To delete a secret

```
kubectl delete secret petstore-secret
```

### Deployment

To deploy the service to a Kubernetes cluster, we need to create a deployment.yaml file with a following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: swagger-petstore-deployment
  labels:
    app: swagger-petstore
spec:
  replicas: 1
  selector:
    matchLabels:
      app: swagger-petstore
  template:
    metadata:
      labels:
        app: swagger-petstore
    spec:
      hostNetwork: true
      containers:
      - name: swagger-petstore
        image: networknt/com.networknt.petstore-2.0.0
        resources:
          limits:
            memory: 512Mi
          requests:
            memory: 256Mi
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        volumeMounts:
        - name: config
          mountPath: "/config"
          readOnly: true
      volumes:
      - name: config
        secret:
          secretName: petstore-secret

```

Note that hostNetwork is used so that the service can register itself to Consul during startup. Also, status.hostIP is passed to the container to get the host IP address. The previously created secret is used in the volumes and volumeMounts. 

To start the service: 

```
kubectl create -f deployment.yaml
```

To delete the service:

```
kubectl delete -f deployment.yaml
```

To list all the running pods:

```
kubectl get pods
```


### Memory Limit

In the above deployment.yaml, we have set the memory limit for the container. To verify that, let's put some load on the server and see if the service respects the limitation. 

To access the server, go to the consul server first to find the IP and port number for the petstore service. Verify if it works with a curl command. 

```
curl -k https://38.113.162.51:2408/v2/pet/111
```

Now use the wrk to generate some load for 5 minutes. 

```
wrk -t1 -c128 -d300s https://38.113.162.51:2408/v2/pet/111
```


To monitor the memory usage, login to the node which petstore is deployed and find the container id with: 

```
docker ps
```

With the id found above, issue the following command to monitor the container memory usage. 

```
docker stats {containerid}
```

You can find that the container only use about 300MB memory with the limit of 512MB


[light-config-test]: https://github.com/networknt/light-config-test
[light-config-prod]: https://github.com/networknt/light-config-prod
[petstore]: https://github.com/networknt/light-config-test/tree/master/light-example-4j/rest/swagger/petstore/kubernetes
