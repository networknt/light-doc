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

Note: This step needs to be verified. 

Logging is very important in microservices architecture as logs must be aggregated in
order to trace all the activities of a particular request from a consumer. We are using
ELK stack for logging. In the above step, Elasticsearch, Logstash and Kibana are all
started in the same docker-compose.yml.

In order for the example-petstore container to forward log files to ELK, we need to
set up log driver on the application container to forward logs to Logstash.
 

Here is the command line to start the example-petstore
 
```
export LOGSTASH_ADDRESS=$(docker inspect --format '{{ .NetworkSettings.Networks.lightdocker_light.IPAddress }}' lightdocker_logstash_1)
docker run -d -p 8443:8443 -v /tmp/petstore/config:/config --network=lightdocker_light --log-driver=gelf --log-opt gelf-address=udp://$LOGSTASH_ADDRESS:12201 --log-opt tag="petstore" --log-opt env=dev networknt/example-petstore

```

Now you example-petstore is up and running and all logs have sent to the Logstash and 
then ElasticSearch. Let's go to the Kibana to see if our logs can be viewed there.

```
http://localhost:5601/
```

Just select the default template and you can see the logs there.
