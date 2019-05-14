---
title: "Módulo Cliente"
date: 2018-03-01T21:33:09-05:00
description: ""
categories: []
keywords: []
weight: 10
aliases: []
toc: false
draft: false
---

O [Módulo Cliente][] em light-4j é um componente muito importante na plataforma light para
interações entre cliente para serviço ou serviço para serviço em requisições/respostas sincronas 
sobre HTTPS e HTTP2. Microserviços são esperado ter alta performance a baixa latência para
que múltiplos serviços podem ser encadeados juntos para cumprir uma requisição externa.
Isso é porque o módulo cliente é desenhado para conexão direta entre consumidor e provedor com o
HTTP 2.0 sobre conexão TLS cacheado no lado do consumidor para maioria de aplicações de alto rendimento.
Para consumidores de baixo rendimento, a conexão pode ser fechada depois da requisição e reaberta quando
a próxima requisição chega.

Há muitas funcionalidades no módulo cliente e esses deveriam ser utilizado em certos cenários.
A seguinte seçao mostra como usar o módulo cliente para casos de uso comum e os detalhes é documentado
aqui também.

- [Conexão TLS](/ptbr/consumer/tls-connection/)
- [HTTP 2.0](/ptbr/consumer/http2/)
- [OAuth 2.0 JWT Token](/ptbr/consumer/oauth2-jwt/)
  * [Token de entidade](/ptbr/consumer/subject-token/)
  * [Token de acesso](/ptbr/consumer/access-token/)
  * [Tipo de concessão personalizado](/ptbr/consumer/customized-grant/)
  * [Obter Token no Startup](/ptbr/consumer/token-startup/)
  * [Token de longa duração](/ptbr/consumer/long-lived-token/)


[Módulo cliente]: /concern/client/