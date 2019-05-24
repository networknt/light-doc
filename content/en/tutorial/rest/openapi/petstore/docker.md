---
title: "Docker"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 80
aliases: []
toc: false
draft: false
---

When petstore is generated, a folder called docker with two default Dockerfiles were
generated at the same time for any further customization. Also, there is a build.sh
in the root folder that can be used to build and publish docker images. 

Here is the content of Dockerfile which uses alpine Linux as base image to get minimum
size. 

```
FROM openjdk:8-jre-alpine
#EXPOSE  8443
ADD /target/petstore-1.0.1.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
``` 

Here is the content of Dockerfile-Redhat which uses Redhat openshift as based image as
some of the customers are using Openshift as Kubernetes cloud provider. 

```
FROM registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift
#EXPOSE  8443
ADD /target/petstore-1.0.1.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar server.jar"]
```  
 
Let's just use the build.sh to create docker images and publish them to docker hub. On
your computer, the publish step will fail as you don't have credential to access networknt
organization on docker hub. 

Make sure you are in light-example-4j/rest/openapi/petstore folder and change the build.sh
to executable. 

```
cd ~/networknt/light-example-4j/rest/openapi/petstore
chmod +x build.sh
build.sh {version}
```

The build.sh will call the maven to build the project again, create docker images and publish
them to docker hub. 


Let's start the docker container.

```
docker run -d -p 8443:8443 networknt/com.networknt.petstore-1.0.1
```

In another terminal, run the curl to access the server.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" https://localhost:8443/v2/pet/111
```

And the result should be:

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

Now let's shutdown the docker container. First use "docker ps" to find the container_id
and then issue docker stop with that container_id.

```
docker ps
docker stop ad86cc533270
```

Note that it takes about 10 seconds to stop the server as in cloud environment, the server
needs to send notification to registry and handle all the in-flight requests until all clients 
have switched to other instances in [client side discovery][]. 

The networknt/com.networknt.petstore-1.0.1 can be found at 
https://hub.docker.com/u/networknt/dashboard/

And the following command can pull and run the docker image on your local if you have
docker installed.

```
docker run -d -p 8443:8443 networknt/com.networknt.petstore-1.0.1
```

Now we have a docker image from public docker hub and it can be started at any computer
with docker installed. However, the configuration files are inside the docker image and
if we want to change any configuration for any middleware handlers in side the docker
contain, we can use externalized config folder for this purpose. In the next step, we are
going to use [docker compose][] with externalized config in light-config-test folder.  

[client side discovery]: /tutorial/common/discovery/
[docker compose]: /tutorial/rest/openapi/petstore/compose/
