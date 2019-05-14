---
title: "SPA Session JWT"
date: 2018-05-01T09:38:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Ao criar SPA que consomem APIs protegidas por tokens JWT ou token simples da Web, muitos artigos sugerem colocar o token no armazenamento local no navegador para que o aplicativo Javascript possa usar o token para acessar as APIs diretamente. ** Por favor, não faça isso, pois não é seguro **.

Como o artigo [pare de usar localstorage] indica, o armazenamento local no navegador não é segurança. Está sujeito a ataques de script entre sites, e as informações podem ser recuperadas por qualquer pessoa que tenha acesso ao navegador fisicamente.

Se você precisar armazenar dados confidenciais, use sempre uma sessão do lado do servidor. Veja como fazer isso:

* Quando um usuário faz login no seu site, crie um identificador de sessão para ele e armazene-o em um cookie assinado criptograficamente. Se você estiver usando uma estrutura da Web, procure "como criar uma sessão de usuário usando cookies" e siga aquele guia.

* Certifique-se de que qualquer biblioteca de cookie usada pelo seu framework web esteja configurando o sinalizador httpOnly cookie. Esse sinalizador torna impossível para um navegador ler qualquer cookie, o que é necessário para usar com segurança as sessões do lado do servidor com cookies. Leia o [artigo do Jeff Atwood][] para mais informações. Ele é o cara.

* Certifique-se de que sua biblioteca de cookies também defina o sinalizador de cookie SameSite = strict (para evitar ataques [CSRF] []), bem como o sinalizador secure = true (para garantir que os cookies só possam ser definidos em uma conexão criptografada).

* Cada vez que um usuário faz uma solicitação ao seu site, use seu ID de sessão (extraído do cookie que ele envia para você) para recuperar os detalhes da conta de um banco de dados ou de um cache (dependendo do tamanho do site)

* Depois que você tiver as informações da conta do usuário acessadas e verificadas, sinta-se à vontade para enviar os dados confidenciais associados.

Esse padrão é simples, direto e, o mais importante: seguro. E sim, você pode definitivamente escalar um grande site usando este padrão ao usar [light-session-4j] [] que é uma sessão distribuída apoiada por uma grade de dados na memória.

[pare de usar localstorage]: https://dev.to/rdegges/please-stop-using-local-storage-1i04
[artigo do Jeff Atwood]: https://blog.codinghorror.com/protecting-your-cookies-httponly/
[CSRF]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)
[light-session-4j]: /ptbr/sytle/light-session-4j/

