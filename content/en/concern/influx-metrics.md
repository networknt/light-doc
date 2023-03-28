---
title: "Metrics"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---


## Introduction

The metrics handler collects the API runtime information and reports it to Influxdb or Broadcom APM periodically (5 minutes to 15 minutes based on the volume of the API). 

For general info about the metrics, please visit [here](/concern/metrics/). 

The InfluxDB MetricsHandler is the first implementation and the default metrics handler in the light-4j frameworks. 

## Configuration

Here is an example of configuration in values.yml


```

# metrics.yml

metrics.enabled: true
metrics.serverHost: localhost
metrics.serverPort: 8086
metrics.serverName: metrics
metrics.serverUser: admin
metrics.serverPass: admin
metrics.reportInMinutes: 1
```

