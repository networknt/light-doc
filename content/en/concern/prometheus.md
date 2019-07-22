---
title: "Prometheus"
date: 2018-02-21T14:11:25-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
---


## Introduction


Prometheus is a tool/database that is used for monitoring. Prometheus adopt a pull based model in getting metrics data by querying each targets defined in its configuration.


Prometheus scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs.
It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts.
Grafana or other API consumers can be used to visualize the collected data.

Light-4j prometheus metrics handler collects the API runtime information and save it to prometheus Metric data module. Prometheus server will pull the metric from metrics_path which is configurable on
prometheus config yml file (for exampe: /v1/prometheus) ; The pulling interval is second base on the config file  (scrape_interval: 30s). A Grafana instance is hooked to Prometheus
to output the metrics on dashboard from two different perspectives.



By default, Prometheus metric targets ( nodes to monitor ) are provided statically in the prometheus config file. Static targets are simple when we know in advance our infrastructure.
But for the microservice architecture, service API will have dynamic target on the cloud. So our solution is use consul to provide targets for prometheus to monitor dynamically


Consul is a service discovery tool by hashicorp, it allows services/virtual machines to be registered to it and then provide dns and http interfaces to query on the state of the registered services/virtual machines.


In prometheus, we need to configure it use consul_sd_targets. Prometheus will then query consul http interface for the catalog of targets to monitor.





## Configuration



#### Configuration file (prometheus.yml) for the MiddlewareHandler (com.networknt.metrics.prometheus.PrometheusHandler)


```

// If metrics handler is enabled or not
enabled: false

 // If the Prometheus hotspot is enabled or not.
// hotspot include thread, memory, classloader,...
enableHotspot: false


```

Change the enabled value to true to enable the prometheus metrics handler;

In the light-4j, the following Prometheus Metric are defined in the Prometheus MiddlewareHandler


```java

    public static final String REQUEST_TOTAL = "requests_total";
    public static final String SUCCESS_TOTAL = "success_total";
    public static final String AUTO_ERROR_TOTAL = "auth_error_total";
    public static final String REQUEST_ERROR_TOTAL = "request_error_total";
    public static final String SERVER_ERROR_TOTAL = "server_error_total";
    public static final String RERSPONSE_TIME_SECOND = "response_time_seconds";

```

And prometheus provides a collector that can scrape and expose the mBeans of a JMX target.

It runs as a JavaAgent, exposing an HTTP server and serving metrics of the local JVM.

If user want to have enable hotspot monitor for JVM, change the enableHotspot value to true:



```
enableHotspot: true
```

The Hotspot monitor metrics include:

CPU and JVM process:

```
process_cpu_seconds_total
process_open_fds
```

JVM memory usage:

```
jvm_memory_pool_allocated_bytes_total
jvm_memory_pool_bytes_init
jvm_memory_pool_bytes_max
jvm_memory_pool_bytes_used
```

metrics about JVM thread areas

```
jvm_threads_peak
jvm_threads_daemon
jvm_threads_started_total
jvm_threads_deadlocked
jvm_threads_state
```

JVM garbage collectors:

```
jvm_gc_collection_seconds
```

Buffer:

```
jvm_buffer_pool_used_bytes
jvm_buffer_pool_capacity_bytes
jvm_buffer_pool_used_buffers
```



## Configuration for Prometheus

Prometheus is configured via a configuration file. The file is written in YAML format, defined by the  Prometheus scheme.



Here is the sample config file. It can be found from link:


```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'services'
    scrape_interval: 15s
    consul_sd_configs:
      - server: 'consul:8500'
        services:
          - 'multimodule_tram-todolist-command-service'
          - 'multimodule_tram-todolist-view-service'
          - 'com.networknt.bookstore-service-api-3.0.1'
          - 'com.networknt.computerstore-service-api-3.0.1'
          - 'com.networknt.foodstore-service-api-3.0.1'
          - 'com.networknt.petstore-service-api-3.0.1'
          - 'com.networknt.marketsample-service-3.0.1'

    tls_config:
      # Skip verification until we have resolved why the certificate validation
      # for the kubelet on API server nodes fail.
      insecure_skip_verify: true

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
    scheme: https

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

    metrics_path: /metrics

```

Currently all we are using https protocol:

     scheme: https

 for local environment, we ignore ssl check by setting in config file:

       insecure_skip_verify: true



## Customization



#### Prometheus Metric names and labels


Every time series is uniquely identified by its metric name and a set of key-value pairs, also known as labels.

The metric name specifies the general feature of a system that is measured (e.g. http_requests_total - the total number of HTTP requests received). It may contain ASCII letters and digits, as well as underscores and colons. It must match the regex [a-zA-Z_:][a-zA-Z0-9_:]*.

Labels enable Prometheus's dimensional data model: any given combination of labels for the same metric name identifies a particular dimensional instantiation of that metric (for example: all HTTP requests that used the method POST to the /api/tracks handler). The query language allows filtering and aggregation based on these dimensions. Changing any label value, including adding or removing a label, will create a new time series.




And the service "endpoint" and "clientId" are been added as Labels for Prometheus's dimensional data model.



The sample result for the Prometheus metric will be like below:


```
requests_total{clientId="unknown",endpoint="/todos@post",instance="172.18.0.6:8081",job="services",service="multimodule_tram-todolist-command-service"}	1
requests_total{clientId="unknown",endpoint="/todoviews@get",instance="172.18.0.7:8082",job="services",service="multimodule_tram-todolist-view-service"}	1

```


## Integrate  Prometheus with Grafana:

Light framwork provided docker-compose file (docker-compose-prometheus.yml) includes the Grafana image: image: grafana/grafana:4.6.3

So user can open browser to view  the Prometheus from Grafana


 ```
http://127.0.0.1:3000/

user: admin
passwork: admin

 ```

Please follow the following step to set the Prometheus in Grafana:

Adding the data source to Grafana:


1. Open the side menu by clicking the Grafana icon in the top header.

2. In the side menu under the Dashboards link you should find a link named Data Sources.

3. Click the + Add data source button in the top header.

4. Select Prometheus from the Type dropdown.

After the Prometheus worked as backend dataSource for the Grafana, user can customize their own template or dashboard in Grafana for Prometheus metrics


You can get the detail from: http://docs.grafana.org/features/datasources/prometheus/