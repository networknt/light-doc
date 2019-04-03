---
title: "Logging"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 100
aliases: []
toc: false
draft: false
---

Logging is very important in microservices architecture as logs must be aggregated in
order to trace all the activities of a particular request from a consumer. We are using
ELK stack for logging. In production in Kubernetes cluster, there are vary options for
logging and we may replace Logstash with Fluentd. In this turorial, Elasticsearch, 
Logstash and Kibana are all started in the same docker-compose.yml.

In order for the example-petstore container to forward log files to ELK, we need to
set up log driver on the application container to forward logs to Logstash and then
to ElasticSearch. 
 
Bofore taking any action, let's first kill the petstore docker container if it is still
running. 
 
For demo purpose, ELK will be started with a docker-compose locally. There is a compose
file available in light-docker called docker-compose-logging.yml

**Please note** that ElasticSearch has some requirements in memory, file handlers etc.
to start up. On my Linux desktop, I need to run the following. 

```
sudo sysctl -w vm.max_map_count=262144
``` 
 
The following is the steps to start ELK locally. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-logging.yml up
```

You can goto http://localhost:9200 to confirm that ElasticSearch is running.

And you can goto http://localhost:5601 to confirm that Kibana is running.

With ELK running, here is the command line to start the petstore docker container. 

### Start Container with Docker
 
```
export LOGSTASH_ADDRESS=$(docker inspect --format '{{ .NetworkSettings.Networks.localnet.IPAddress }}' lightdocker_logstash_1)
docker run -d -p 8443:8443 --network=localnet --log-driver=gelf --log-opt gelf-address=udp://$LOGSTASH_ADDRESS:12201 --log-opt tag="petstore" --log-opt env=dev networknt/com.networknt.petstore-1.0.1
```

Now your petstore is up and running and all logs have sent to the Logstash and 
then ElasticSearch. Let's go to the Kibana to see if our logs can be viewed there.

```
http://localhost:5601/
```

Just leave the default Index pattern (logstash-*) and select @timestamp for Time Filter
field name then click Create button. 
 
Now you can see the logs if you hit Discover menu. 


### Start Container with Compose

First let's stop the docker container by finding the container id first with "docker ps"
and then issue a docker stop {container id}

As we have export the LOGSTASH_ADDRESS in the previous step, don't need to do that again.
If you have skip the previous step, then you need to do the export. 

```
export LOGSTASH_ADDRESS=$(docker inspect --format '{{ .NetworkSettings.Networks.localnet.IPAddress }}' lightdocker_logstash_1)
```

Once the environment variable is set, let's start the server with docker-compose.

```
cd ~/networknt/light-config-test/light-example-4j/rest/openapi/petstore/logging
docker-compose up
```

Once the docker-compose is up, you can goto Kibana to view the logs. 

```
http://localhost:5601/
```

As you can see this time each log entry has tag as "petstore-compose" to indicate that it
is collected from the docker-compose. 

Here is one example entry. 

```
March 10th 2018, 17:44:24.714	source_host:172.18.0.1 level:6 created:March 10th 2018, 17:44:22.136 message:Https Server started on ip:0.0.0.0 Port:8443 version:1.1 command:/bin/sh -c java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar image_name:networknt/com.networknt.petstore-1.0.1 @timestamp:March 10th 2018, 17:44:24.714 container_name:logging_petstore_1 host:joy @version:1 tag:petstore-compose image_id:sha256:8013f49a287c998886f5fdc65c18bd62a4674d06bd4d7d111345272f716b5fe1 container_id:c3fba2f3c9f3900d75896ad91f7b0fd2a18757b411cf2f7462b41fdb031ed165 _id:AWISFigbs8Bk8xA0uV2x _type:logs _index:logstash-2018.03.10 _score: -
```

Above steps are just demos that give you an idea how centralized logging looks like. They
are not supposed to be used for production. Production with Kubernetes will be covered in
another tutorial. 

In the next step, we are going to enable [metrics][] to InfluxDB. 

[metrics]: /tutorial/rest/openapi/petstore/metrics/
