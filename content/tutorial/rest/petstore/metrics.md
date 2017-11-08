---
title: "Metrics"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 90
aliases: []
toc: false
draft: false
---

Note: This step needs to be verified.

In order to use oauth2(light-oauth2), metrics(Influxdb and Grafana) and 
logging(Elasticsearch, Logstash and Kibana), we've cloned the light-docker repo.


Now let's start all services defined in docker-compose.yml

```
cd ~/networknt/light-docker
docker-compose up --build
```

First let's make sure that Influxdb and Grafana can be accessed.

```
http://localhost:8083/
```
Make sure Influxdb admin page is shown up and metrics database is created.

```
http://localhost:3000/
```

Make sure Grafana dashboard is up and you can login with admin/admin.

Once both Influxdb and Grafana are up and running, let's stop the example-petstore
container by issuing "docker ps" on another terminal to find out the container_id of
example-petstore. Note: there are several docker containers running so double check
you have picked the right container_id.

Now run the following command to stop the example-petstore

```
docker stop [container_id of example-petstore]
```

In the next step, we are going to start the same container with externalized metrics
config so that the server can connect to the Influxdb container to report the metrics.

Let's create a folder under /tmp and name it petstore. Within /tmp/petstore, create
another folder called config. Now create metrics.yml in /tmp/petstore/config folder.

```
description: Metrics handler configuration
enabled: true
influxdbProtocol: http
influxdbHost: influxdb
influxdbPort: 8086
influxdbName: metrics
influxdbUser: root
influxdbPass: root
reportInMinutes: 1
```

Please note that the above configuration is only for testing. 

Now start the container and linked to Influxdb. 

```
docker run -d -p 8443:8443 -v /tmp/petstore/config:/config --network=lightdocker_light networknt/example-petstore
```
Access the one endpoint several times with curl command and wait for one minute.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" https://localhost:8443/v2/pet/111
```

Go to http://localhost:8083/ and select metrics database and select "SHOW MEASUREMENTS". 
You will find several measurements created. Some of them are for api view and some of them
are for client view. 

You can connect Grafana to Influxdb and create dashboards on Grafana to visualize
the metrics. 

