---
title: "Token de entidade"
date: 2018-03-01T21:37:35-05:00
description: ""
categories: []
keywords: []
weight: 50
aliases: []
toc: false
draft: false
---

Conforme descrito em [arquitetura de segurança] [], a plataforma light usa dois tokens para proteger o serviço
para atender chamadas. Um token é o token original do usuário de login e tem original
informações do chamador e possíveis informações adicionais para autorização de baixa granularidade. Este token
é chamado Token de entidade.

O token de entidade representa uma pessoa e tem user_id e informações adicionais do usuário em potencial.

Há duas maneiras para obter esse token.

* Fluxo de concessão de autorização para o provedor do OAuth 2.0

* OpenID Connect

Este token é passado no cabeçalho de solicitação "Authorization" na maioria dos casos e se o token
contém informação de escopo para acessar a API/serviço, então nenhum Token de Acesso é necessário. este
é o servidor web da situação como um cliente chamando APIs.

This token is passed in request header "Authorization" in most of the cases and if the token
contains scope info to access the immediate API/service, then no Access Token is needed. This
is the situation webserver as a client calling APIs.


[security architecture]: /architecture/security/
[Access Token]: /ptbr/consumer/access-token/

