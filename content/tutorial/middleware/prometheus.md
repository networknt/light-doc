---
title: "How to use Prometheus for metrics"
date: 2018-02-21T14:15:14-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


## Introduction


Light-4j provide  Prometheus metrics handler as MiddlewareHandler to collects the API runtime information and save it to Prometheus Metric data module. And then use pre-defined HttpHandler (PrometheusGetHandler) to expose the prometheus Metric data module;
Prometheus server will pull the metric from metrics_path which is configurable on

Prometheus config yml file (for exampe: /v1/prometheus) ; The pulling interval is second base on the config file  (scrape_interval: 30s).



## Configuration

#### Configuration file (prometheus.yml) for the MiddlewareHandler (com.networknt.metrics.prometheus.PrometheusHandler)

```
# Metrics handler configuration

# If prometheus metrics handler is enabled or not
enabled: false

```

Change the enabled value to true to enable the Prometheus metrics handler;


## Configuration for services

If user want to use Prometheus metrics handler as metrics MiddlewareHandler handler for the service, simply set the config "prometheusMetrics" as true

#### Config to generate new service by light-codegen

Here is an example of config.json for openapi generator.

```
{
  "name": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 8080,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false,
  "prometheusMetrics": true
}
```

Then following the step to [generate service API][].

The generated service will use Prometheus metrics handler as metrics MiddlewareHandler handler and the the API runtime information will be save to prometheus Metric data module.




#### Change existing service to use light-4j Prometheus  MiddlewareHandler handler


If user want to change existing service which created before based on light-4j palteform (default use  [InfluxDB metrics handler]: /concern/metrics/) to use Prometheus metrics handler, please change the service by following steps:


1.  Change the service POM to add light-4j Prometheus  MiddlewareHandler handler dependency:


```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>prometheus</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```


2.  Change the service.yml file to use Prometheus  MiddlewareHandler handler:

```
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.prometheus.PrometheusHandler
  ...

```


3.  Change the api spec (openapi.json or swagger.json) by adding Prometheus metrics_path:

```
"/prometheus":{"get":{"responses":{"200":{"description":"successful operation"}},"parameters":[]}}
```


4. Change the


```java
import com.networknt.metrics.prometheus.PrometheusGetHandler;

public class PathHandlerProvider implements HandlerProvider {
    @Override
    public HttpHandler getHandler() {
        return Handlers.routing()
                .add(Methods.GET, "/v1/prometheus", new PrometheusGetHandler());
    }
}

```

The metrics_path is configurable. In light-codegen, we set it as /prometheus, but it can be change to any url user want. Then user just to to set the same "metrics_path" in the Prometheus config file.




## Configuration for Prometheus (config with Consul )

Prometheus is configured via a configuration file. The file is written in YAML format, defined by the  Prometheus scheme. By default, the targets ( nodes to monitor ) are provided statically in the prometheus config file. Static targets are simple when we know in advance our infrastructure.
But for the microservice architecture, service API will have dynamic target on the cloud. So our solution is use consul to provide targets for prometheus to monitor dynamically


In prometheus, we need to configure it use consul_sd_targets. Prometheus will then query consul http interface for the catalog of targets to monitor.



Here is the sample config file. It can be found from link:


```
  - job_name: 'services'
    scrape_interval: 15s
    consul_sd_configs:
      - server: 'consul:8500'
        services:
          - 'multimodule_tram-todolist-command-service'
          - 'multimodule_tram-todolist-view-service'

    relabel_configs:
      - source_labels: ['__meta_consul_service']
        regex:         '(.*)'
        target_label:  'service'
        replacement:   '$1'
      - source_labels: ['__meta_consul_tags']
        target_label: 'slave_name'
        regex: '.*slave_name=([^,]*).*'
        replacement: '${1}'
      - source_labels: ['__meta_consul_tags']
        target_label: 'public_hostname'
        regex: '.*public_hostname=([^,]*).*'
        replacement: '${1}'

    metrics_path: /v1/prometheus
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.


```



## Verification:

1. The latest version of light-tram-4j example todo-list has been change to use prometheus metrics handler; Please follow the step in the tutrial to start the sample service api:

[light-tram todo-list service API][]





2. Starts consul service discovery by docker-compose:

 ```

 cd ~/networknt/light-docker
 docker-compose -f docker-compose-consul.yml up

 ```


3. Starts registrator service by docker-compose:

 ```

 cd ~/networknt/light-docker
 docker-compose -f docker-compose-registrator.yml up

 ```


4. Starts Prometheus server by docker-compose (the Prometheus server will start based on the config on ~/networknt/light-docker/prometheus/prometheus.yml. User can change it for different api):

 ```

 cd ~/networknt/light-docker
 docker-compose -f docker-compose-prometheus.yml up

 ```

5. Open a browser to verify Consul:

 ```
http://127.0.0.1:8500/

 ```

6. Open a browser to verify Prometheus:

 ```
http://localhost:9090/

 ```


 [generate service API]: /tutorial/generator/openapi/
 [light-tram todo-list service API]: /tutorial/tram/todo-list/
