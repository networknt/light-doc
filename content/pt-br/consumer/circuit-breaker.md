---
title: "Circuit Breaker"
date: 2019-04-25T14:58:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Circuit breaker pode ser utilizado através do método Http2Client::getRequestService, isto utiliza uma requisição envolvida por um produtor de java.util.concurrent.CompletableFuture, ou seja, toda vez que executa CircuitBreaker::call, cria uma nova requisição e executa a requisição.

Há uma configuração específica no client.yml, exemplo:

```yml
request:
  errorThreshold: 5
  timeout: 3000
  resetTimeout: 600000
```

* errorThreshold: quantidade de errors de timeout antes de mudar o estado de aberto (valor padrão: 5).
* timeout: quantidade de tempo em milisegundos para esperar pela resposta de uma requisição (valor padrão: 5000)
* resetTimeout: quantidade de tempo em milisegundos para manter estado aberto antes de mudar para meio aberto (valor padrão: 60000).

se não é provido um valor para qualquer parâmetro, é adotado valor padrão.