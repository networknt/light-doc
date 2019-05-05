---
title: "Access Token"
date: 2018-03-01T21:37:48-05:00
description: ""
categories: []
keywords: []
weight: 60
aliases: []
toc: false
draft: false
---

Access token é utilizado para invocação de serviço a serviço ou aplicação standalone que invoca um serviço.
Geralmente, este token é recebido de um provedor OAuth2 2.0 através do fluxo de permissionamento de 
credenciais do cliente.
O token não tem qualquer informação sobre o usuário mas somente tem client_id para indicar quem é a 
aplicação que invocou.

Access token é muito importante para escopos de serviço que invoca como [Token de entidade][],
na cadeia de invocação não terá escopos para todos os serviços.

Se access token é somente o token disponível, isso é passado em Authorization header caso contrário, 
isso é passado em um X-Scope-Token na requisição.


[Token de entidade]: /consumer/subject-token/
