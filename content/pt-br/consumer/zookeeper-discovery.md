---
title: "Zookeeper Discovery"
date: 2018-03-01T21:40:40-05:00
description: ""
categories: []
keywords: []
weight: 200
aliases: []
toc: false
draft: false
---

O Apache Zookeeper pode ser usado como registro de serviço e descoberta com o módulo [zookeeper][] na light-4j; no entanto, não é recomendado, pois é muito pesado e lento. Ele só deve ser usado quando você já tiver o cluster implementado em casa e o número de serviços em execução for pequeno. Caso contrário, recomenda-se usar [Consul][] para registro e descoberta de serviço.

[zookeeper]: /concern/zookeeper/
[Consul]: /ptbr/consumer/consul-discovery/
