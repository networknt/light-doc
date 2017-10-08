---
date: 2017-09-24T14:38:50-04:00
title: Infrastructure Setup
---

The following components are for infrastructure to services built on top of light-*-4j frameworks.

## For development

The development, a [docker-compose](https://github.com/networknt/light-docker/blob/master/docker-compose-api.yml) 
is recommended as it is very easy to get environment setup. 

## For production

* [messaging](https://networknt.github.io/light-4j/devops/messaging/) - Confluent Platform(Kafka and Zookeeper)
* [logging](https://networknt.github.io/light-4j/devops/logging/) - Elastic Search, Logstash and Kabana
* [metrics](https://networknt.github.io/light-4j/devops/metrics/) - InfluxDB, Grafana
* [discovery](https://networknt.github.io/light-4j/devops/discovery/) - Consul
* [database](https://networknt.github.io/light-4j/devops/database/) - Mysql and ArangoDB
* [security](https://networknt.github.io/light-4j/devops/security/) - light-oauth2
* [eventuate](https://networknt.github.io/light-4j/devops/eventuate/) - Eventual consistency framework
* [portal](https://networknt.github.io/light-4j/devops/portal/) - Service management and marketplace.
* [jenkins](https://networknt.github.io/light-4j/devops/jenkins/) - Build and release service.
* [email](https://networknt.github.io/light-4j/devops/email/) - Email server
