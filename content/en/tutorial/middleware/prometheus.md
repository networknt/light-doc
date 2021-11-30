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

# If the Prometheus hotspot is enabled or not.
# hotspot include thread, memory, classloader,...
enableHotspot: false

```

Change the enabled value to true to enable the Prometheus metrics handler; And if we need thread, memory, classloader hotspots metrics information, enable hotspots as well.



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


2.  Change the handler.yml file to use Prometheus  MiddlewareHandler handler:

```
handlers:
  - com.networknt.metrics.prometheus.PrometheusHandler@prometheus
  - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  
chains:
  default:
    - exception
    #- metrics
    - prometheus  
  
  - path: '/prometheus'
    method: 'get'
    exec:
      - getprometheus  
```

3 . add an endpoint for prometheus metrics:

```text
  - path: '/prometheus'
    method: 'get'
    exec:
      - getprometheus
```


4.  Change the api spec (openapi.json or swagger.json) by adding Prometheus metrics_path:

```
"/prometheus":{"get":{"responses":{"200":{"description":"successful operation"}},"parameters":[]}}
```




The metrics_path is configurable. In light-codegen, we set it as /prometheus, but it can be change to any url user want. Then user just to to set the same "metrics_path" in the Prometheus config file.

After the API server start, the prometheus metrics can get by path /prometheus

http://localhost:8080/prometheus

Below is the sample prometheus report:
```text
# HELP jvm_classes_loaded The number of classes that are currently loaded in the JVM
# TYPE jvm_classes_loaded gauge
jvm_classes_loaded 5201.0
# HELP jvm_classes_loaded_total The total number of classes that have been loaded since the JVM has started execution
# TYPE jvm_classes_loaded_total counter
jvm_classes_loaded_total 5201.0
# HELP jvm_classes_unloaded_total The total number of classes that have been unloaded since the JVM has started execution
# TYPE jvm_classes_unloaded_total counter
jvm_classes_unloaded_total 0.0
# HELP requests_total requests_total
# TYPE requests_total counter
requests_total{endpoint="/campsite@get",clientId="unknown",} 1.0
requests_total{endpoint="/campsite@post",clientId="unknown",} 1.0
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 9.640625
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.636848477162E9
# HELP jvm_gc_collection_seconds Time spent in a given JVM garbage collector in seconds.
# TYPE jvm_gc_collection_seconds summary
jvm_gc_collection_seconds_count{gc="G1 Young Generation",} 3.0
jvm_gc_collection_seconds_sum{gc="G1 Young Generation",} 0.034
jvm_gc_collection_seconds_count{gc="G1 Old Generation",} 0.0
jvm_gc_collection_seconds_sum{gc="G1 Old Generation",} 0.0
# HELP jvm_info JVM version info
# TYPE jvm_info gauge
jvm_info{version="11.0.11+9-LTS-194",vendor="Oracle Corporation",runtime="Java(TM) SE Runtime Environment",} 1.0
# HELP jvm_buffer_pool_used_bytes Used bytes of a given JVM buffer pool.
# TYPE jvm_buffer_pool_used_bytes gauge
jvm_buffer_pool_used_bytes{pool="mapped",} 1.38154803E8
jvm_buffer_pool_used_bytes{pool="direct",} 131072.0
# HELP jvm_buffer_pool_capacity_bytes Bytes capacity of a given JVM buffer pool.
# TYPE jvm_buffer_pool_capacity_bytes gauge
jvm_buffer_pool_capacity_bytes{pool="mapped",} 1.38154803E8
jvm_buffer_pool_capacity_bytes{pool="direct",} 131072.0
# HELP jvm_buffer_pool_used_buffers Used buffers of a given JVM buffer pool.
# TYPE jvm_buffer_pool_used_buffers gauge
jvm_buffer_pool_used_buffers{pool="mapped",} 1.0
jvm_buffer_pool_used_buffers{pool="direct",} 8.0
# HELP response_time_seconds response_time_seconds
# TYPE response_time_seconds summary
response_time_seconds_count{endpoint="/campsite@get",clientId="unknown",} 1.0
response_time_seconds_sum{endpoint="/campsite@get",clientId="unknown",} 0.1250344
response_time_seconds_count{endpoint="/campsite@post",clientId="unknown",} 1.0
response_time_seconds_sum{endpoint="/campsite@post",clientId="unknown",} 0.0829096
# HELP jvm_memory_bytes_used Used bytes of a given JVM memory area.
# TYPE jvm_memory_bytes_used gauge
jvm_memory_bytes_used{area="heap",} 7.477728E7
jvm_memory_bytes_used{area="nonheap",} 4.8414648E7
# HELP jvm_memory_bytes_committed Committed (bytes) of a given JVM memory area.
# TYPE jvm_memory_bytes_committed gauge
jvm_memory_bytes_committed{area="heap",} 4.00556032E8
jvm_memory_bytes_committed{area="nonheap",} 5.1970048E7
# HELP jvm_memory_bytes_max Max (bytes) of a given JVM memory area.
# TYPE jvm_memory_bytes_max gauge
jvm_memory_bytes_max{area="heap",} 6.400507904E9
jvm_memory_bytes_max{area="nonheap",} -1.0
# HELP jvm_memory_bytes_init Initial bytes of a given JVM memory area.
# TYPE jvm_memory_bytes_init gauge
jvm_memory_bytes_init{area="heap",} 4.00556032E8
jvm_memory_bytes_init{area="nonheap",} 7667712.0
# HELP jvm_memory_pool_bytes_used Used bytes of a given JVM memory pool.
# TYPE jvm_memory_pool_bytes_used gauge
jvm_memory_pool_bytes_used{pool="CodeHeap 'non-nmethods'",} 1243776.0
jvm_memory_pool_bytes_used{pool="Metaspace",} 3.5675432E7
jvm_memory_pool_bytes_used{pool="CodeHeap 'profiled nmethods'",} 6199552.0
jvm_memory_pool_bytes_used{pool="Compressed Class Space",} 3837840.0
jvm_memory_pool_bytes_used{pool="G1 Eden Space",} 6.815744E7
jvm_memory_pool_bytes_used{pool="G1 Old Gen",} 4522688.0
jvm_memory_pool_bytes_used{pool="G1 Survivor Space",} 2097152.0
jvm_memory_pool_bytes_used{pool="CodeHeap 'non-profiled nmethods'",} 1458048.0
# HELP jvm_memory_pool_bytes_committed Committed bytes of a given JVM memory pool.
# TYPE jvm_memory_pool_bytes_committed gauge
jvm_memory_pool_bytes_committed{pool="CodeHeap 'non-nmethods'",} 2555904.0
jvm_memory_pool_bytes_committed{pool="Metaspace",} 3.6438016E7
jvm_memory_pool_bytes_committed{pool="CodeHeap 'profiled nmethods'",} 6225920.0
jvm_memory_pool_bytes_committed{pool="Compressed Class Space",} 4194304.0
jvm_memory_pool_bytes_committed{pool="G1 Eden Space",} 7.4448896E7
jvm_memory_pool_bytes_committed{pool="G1 Old Gen",} 3.24009984E8
jvm_memory_pool_bytes_committed{pool="G1 Survivor Space",} 2097152.0
jvm_memory_pool_bytes_committed{pool="CodeHeap 'non-profiled nmethods'",} 2555904.0
# HELP jvm_memory_pool_bytes_max Max bytes of a given JVM memory pool.
# TYPE jvm_memory_pool_bytes_max gauge
jvm_memory_pool_bytes_max{pool="CodeHeap 'non-nmethods'",} 5898240.0
jvm_memory_pool_bytes_max{pool="Metaspace",} -1.0
jvm_memory_pool_bytes_max{pool="CodeHeap 'profiled nmethods'",} 1.2288E8
jvm_memory_pool_bytes_max{pool="Compressed Class Space",} 1.073741824E9
jvm_memory_pool_bytes_max{pool="G1 Eden Space",} -1.0
jvm_memory_pool_bytes_max{pool="G1 Old Gen",} 6.400507904E9
jvm_memory_pool_bytes_max{pool="G1 Survivor Space",} -1.0
jvm_memory_pool_bytes_max{pool="CodeHeap 'non-profiled nmethods'",} 1.2288E8
# HELP jvm_memory_pool_bytes_init Initial bytes of a given JVM memory pool.
# TYPE jvm_memory_pool_bytes_init gauge
jvm_memory_pool_bytes_init{pool="CodeHeap 'non-nmethods'",} 2555904.0
jvm_memory_pool_bytes_init{pool="Metaspace",} 0.0
jvm_memory_pool_bytes_init{pool="CodeHeap 'profiled nmethods'",} 2555904.0
jvm_memory_pool_bytes_init{pool="Compressed Class Space",} 0.0
jvm_memory_pool_bytes_init{pool="G1 Eden Space",} 2.7262976E7
jvm_memory_pool_bytes_init{pool="G1 Old Gen",} 3.73293056E8
jvm_memory_pool_bytes_init{pool="G1 Survivor Space",} 0.0
jvm_memory_pool_bytes_init{pool="CodeHeap 'non-profiled nmethods'",} 2555904.0
# HELP success_total success_total
# TYPE success_total counter
success_total{endpoint="/campsite@get",clientId="unknown",} 1.0
success_total{endpoint="/campsite@post",clientId="unknown",} 1.0
# HELP jvm_threads_current Current thread count of a JVM
# TYPE jvm_threads_current gauge
jvm_threads_current 27.0
# HELP jvm_threads_daemon Daemon thread count of a JVM
# TYPE jvm_threads_daemon gauge
jvm_threads_daemon 7.0
# HELP jvm_threads_peak Peak thread count of a JVM
# TYPE jvm_threads_peak gauge
jvm_threads_peak 27.0

```

## Local Configuration for Prometheus:

```text
#Global configurations
global:
  scrape_interval:     5s # Set the scrape interval to every 5 seconds.
  evaluation_interval: 5s # Evaluate rules every 5 seconds.

scrape_configs:
  - job_name: 'campsite-app'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['campsite-service:8080']
```


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



## Using Prometheus in Grafana:

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

You can get the detail from: http://docs.grafana.org/features/datasources/prometheus/
