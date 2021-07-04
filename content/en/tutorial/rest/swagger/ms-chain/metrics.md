---
title: "Metrics"
date: 2017-11-29T07:09:05-05:00
description: "Enable Metrics"
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

All services can be configured to enable metrics collection and report metrics info to Influxdb 
and subsequently viewed from Grafana dashboard. 

If InfluxDB is not available the report will be a noop.

In order to output to the right Influxdb instance, we need to create metrics.yml in
src/main/resources/config folder to overwrite the default configuration. 


Let's create a new folder metrics by copying the security folder.

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a
cp -r security metrics
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b
cp -r security metrics
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c
cp -r security metrics
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d
cp -r security metrics

```

Now we need to add the following metrics.yml to each api resources/config.

```
# Metrics handler configuration

# If metrics handler is enabled or not
enabled: true

# influxdb protocal can be http, https
influxdbProtocol: http
# influxdb hostname
influxdbHost: localhost
# influxdb port number
influxdbPort: 8086
# influxdb database name
influxdbName: metrics
# influxdb user
influxdbUser: root
# influx db password
influxdbPass: root
# report and reset metrics in minutes.
reportInMinutes: 1

```

Now let's start Influxdb and Grafana from docker-compose-metrics.yml in light-docker.
The light-docker repo should have been checked out at preparation step.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-metrics.yml up
```

Now let's start four APIs from four terminals. 

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d/metrics
mvn clean install exec:exec

```

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/metrics
mvn clean install exec:exec
```

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b/metrics
mvn clean install exec:exec
```

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/metrics
mvn clean install exec:exec
```


Let's use curl to access API A, this time I am using a long lived token I generated 
from a utility.

```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgwNjIwMDY2MSwianRpIjoibmQtb2ZZbWRIY0JZTUlEYU50MUFudyIsImlhdCI6MTQ5MDg0MDY2MSwibmJmIjoxNDkwODQwNTQxLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.SPHICXRY4SuUvWf0NYtwUrQ2-N-NeYT3b4CvxbzNl7D7GL5CF91G3siECrRBVexe0smBHHeiP3bq65rnCVFtwlYYqH6ZS5P7-AFiNcLBzSI9-OhV8JSf5sv381nk2f41IE4av2YUlgY0_mcIDo24ItnuPCxj0l49CAaLb7b1SHZJBQJANJTeQj-wgFsEqwafA-2wH2gehtH8CmOuuYfWO5t5IehP-zJNVT66E4UTRfvvZaJIvNTEQBWPpaZeeK6e56SyBqaLOR7duqJZ8a2UQZRWsDdIVt2Y5jGXQu1gyenIvCQbYLS6iglg6Xaco9emnYFopd2i3psathuX367fvw

```

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgwNjIwMDY2MSwianRpIjoibmQtb2ZZbWRIY0JZTUlEYU50MUFudyIsImlhdCI6MTQ5MDg0MDY2MSwibmJmIjoxNDkwODQwNTQxLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.SPHICXRY4SuUvWf0NYtwUrQ2-N-NeYT3b4CvxbzNl7D7GL5CF91G3siECrRBVexe0smBHHeiP3bq65rnCVFtwlYYqH6ZS5P7-AFiNcLBzSI9-OhV8JSf5sv381nk2f41IE4av2YUlgY0_mcIDo24ItnuPCxj0l49CAaLb7b1SHZJBQJANJTeQj-wgFsEqwafA-2wH2gehtH8CmOuuYfWO5t5IehP-zJNVT66E4UTRfvvZaJIvNTEQBWPpaZeeK6e56SyBqaLOR7duqJZ8a2UQZRWsDdIVt2Y5jGXQu1gyenIvCQbYLS6iglg6Xaco9emnYFopd2i3psathuX367fvw" https://localhost:7441/v1/data
```

Send several request from curl and let's take a look at influxdb console at
http://localhost:8083

On the console, select the database to metrics and select from Query templates with 
"SHOW MEASUREMENTS" and you can see something like the following picture.

![influx_console](/images/influx_console.png)

Pick one of the measurement and create a query and you can see the content of the table.

```
select * from "com.networknt.apia-1.0.0.request.count"
```

![api_a_metrics](/images/api_a_metrics.png)


If you want to take a look at metrics from one of client's perspective, issue the following query

```
select * from "f7d42348-c647-4efb-a52d-4c5787421e72.request.count"
```

![client_metrics](/images/client_metrics.png)

