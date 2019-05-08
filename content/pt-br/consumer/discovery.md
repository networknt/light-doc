---
title: "Descoberta de serviços"
date: 2018-03-01T21:40:21-05:00
description: ""
categories: []
keywords: []
weight: 180
aliases: []
toc: false
draft: false
---
Quando implantar microserviços para um cluster kubernete na nuvem, há nenhum IP estático e número de porta para esses 
serviços. Para invocar esses serviços, ou você usa proxies provido por seu fornecedor de cluster kubernetes ou manipula
isso com serviço de registro da plataforma light e descoberta na rede do host. 

Fizemos um teste de desempenho com um de nossos clientes no cluster Openhift e encontramos os seguintes números interessantes com um dos serviços criados sobre o light-rest-4j.

### Uso de proxies Openshift

* Taxa de transferência: 2450
* Latência: 68ms

### Uso de descoberta de serviço Consul

* Taxa de transferência: 20000
* Latência: 6ms

Como você pode ver, quando os proxies são usados no Openshift, todo o cluster só pode ter um total de aproximadamente 2.500 solicitações por segundo. Imagine que você tenha centenas de serviços compartilhando o mesmo throughput, então cada serviço terá apenas várias solicitações por segundo.

Acima está a razão pela qual temos que fornecer nosso próprio registro de serviço e descoberta para que nossos serviços possam se comunicar entre si sem passar por qualquer proxy. Isso significa que não precisamos de nenhum salto extra na rede. Como temos conexão direta com o HTTP 2.0 que suporta multiplexação, podemos cachear a conexão por alguns minutos ou várias solicitações para acelerar ainda mais a interação dos serviços.

### Resumo

Há vários produtos de código aberto que suportam registro de serviço e descoberta no mercado. Nós implementamos o módulo em light-4j para suportar tanto [Consul] [] quanto [Zookeeper] [].

[Consul]: /consumer/consul-discovery/
[Zookeeper]: /consumer/zookeeper-discovery/
