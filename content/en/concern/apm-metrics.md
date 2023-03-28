---
title: "Apm Metrics"
date: 2023-03-16T16:23:41-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We have some customers who are using Broadcom APM for metrics handling and presentation. This AMPMetricsHandler is a customized one that can connect to the EPAgent deployed on the Kubernetes cluster as a common service or VM host for VM deployment. 

The APM Agent support RESTful Interface that is defined [here](https://techdocs.broadcom.com/us/en/ca-enterprise-software/it-operations-management/application-performance-management/10-7/implementing-agents/infrastructure-agent/epagent-plug-ins/configure-epagent-network-ports/enable-the-epagent-plug-ins-restful-interface.html)


The metrics collection is managed by the AbstractMetricsHandler and both push metrics handlers ([MetricsHandler][] and APMMetricsHandler are extended from the AbstractMetricsHandler.


For general information on the metrics, please visit [here](/concern/metrics/).

### Configuration

By default, InfluxDB MetricsHandler is used in the default chain generated. We need to replace it with the APMMetricsHandler in the handlers definition section in the handler.yml file.

handler.yml

```
handlers:
  .
  .
  .
  - com.networknt.metrics.APMMetricsHandler@metrics
  .
  .
  .
chain.default
  .
  .
  .
  - metrics
  .
  .
  .
```

In the values.yml file, we need to overwrite some properties from the metrics.yml file. 

values.yml

```
# metrics.yml
metrics.serverHost: opentracing.ccaapm.svc.cluster.local
metrics.serverPort: 8888
metrics.serverPath: /apm/metricFeed
metrics.productName: http-sidecar
metrics.sendIssuer: true
metrics.issuerRegex: /([^/]+)$

```

If you want to capture the outbound and inbound request time for proxy handler and router handler with http-sidecar and light-gateway, you can enable that with the following in values.yml file.

```
# proxy.yml
proxy.metricsInjection: true

# router.yml
router.metricsInjection: true
```

