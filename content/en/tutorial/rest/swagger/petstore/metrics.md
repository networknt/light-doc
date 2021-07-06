---
title: "Metrics"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 100
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have explored how to send logs to a centralized server so that activity across multiple services can be viewed on the same screen. In this step, we are going to setup metrics in the pet store to send it to InfluxDB and then subsequently viewed the dashboard from Chronograf UI.

In order to start InfluxDB, Chronograf and Kapacitor, a docker-compose file will be used from light-docker repository. 

Now let's start all services defined in docker-compose-metrics.yml

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-metrics.yml up
```

First let's make sure that InfluxDB and Chronograf can be accessed.

```
http://localhost:8888/
```

Make sure Chronograf admin page is shown up and connect to metrics database.

You need to change the url from http://localhost:8086 to http://influxdb:8086 and
set the database name to metrics. 

user is root and password is root as well. 

Once both Influxdb and Chronograf are up and running, let's stop the swagger petstore
container by issuing "docker ps" on another terminal to find out the container_id of
petstore. Note: there are several docker containers running so double check you have 
picked the right container_id.

Now run the following command to stop the example-petstore

```
docker stop [container_id of petstore]
```

In the next step, we are going to start the same container with externalized metrics
config so that the server can connect to the Influxdb container to report the metrics.

To make is simple, we are going to use docker-compose from light-config-test 

There is a folder at ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/metrics
with security.yml and metrics.yml

Here is the content of metrics.yml

```
enabled: true
influxdbProtocol: http
influxdbHost: influxdb
influxdbPort: 8086
influxdbName: metrics
influxdbUser: root
influxdbPass: root
reportInMinutes: 1
```

The security.yml has enableVerifyJwt set as true. 

Please note that the above configuration is only for testing. 

Now start the docker-compose 

```
cd ~/networknt/light-config-test/light-example-4j/rest/swagger/petstore/metrics
docker-compose up
```
Access the one endpoint several times with curl command and wait for one minute.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" https://localhost:8443/v2/pet/111
```

Go to http://localhost:8888/ and select metrics database and select "SHOW MEASUREMENTS". 
You will find several measurements created. Some of them are for api view and some of them
are for client view. 

You can use Grafana to connect to InfluxDB and create dashboards on Grafana to visualize
the metrics in production. 

In this step, we have enabled metrics so that we can have a centralized metrics connection
at runtime. 


