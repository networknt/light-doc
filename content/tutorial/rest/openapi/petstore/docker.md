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

When petstore is generated, a default Dockerfile is there ready for any further 
customization. 
 
Let's just use it to create a docker image and start a docker container. Make sure you
are in light-example-4j/rest/openapi/petstore folder.


```
cd ~/networknt/light-example-4j/rest/openapi/petstore
docker build -t networknt/openapi-petstore .
```

Let's start the docker container.

```
docker run -d -p 8443:8443 networknt/openapi-petstore
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

The next step, let's push the docker image to docker hub. This assumes that you have
an account on docker hub. For me, I am going to push it to networknt/openapi-petstore.

Please skip this step if you don't have a docker hub account yet.

```
docker images
docker tag 9f0b9fe29c44 networknt/openapi-petstore:latest
docker push networknt/openapi-petstore
```

The example-petstore can be found at https://hub.docker.com/u/networknt/dashboard/

And the following command can pull and run the docker image on your local if you have
docker installed.

```
docker run -d -p 8443:8443 networknt/openapi-petstore
```
