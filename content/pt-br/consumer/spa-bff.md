---
title: "Single Page App Proxy"
date: 2018-05-13T23:52:10-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Ao desenvolver o SPA React com uma estrutura do lado do cliente como o React, é aconselhável aproveitar o WebPack hot loader para obter maior produtividade. Isso requer que iniciemos um servidor Nodejs para servir o SPA ao navegador. Quando o SPA está chamando algumas das APIs, os servidores da API podem estar em outro host ou no mesmo host, mas com um número de porta diferente. Para garantir que o aplicativo Javascript precise especificar apenas o caminho relativo sem se preocupar com o host e a porta, precisamos configurar um proxy no arquivo package.json gerado.

A seguir é um exemplo que nós estamos utilizando para desenvolvimento.

```
proxy: {
  "/v1": {
    target: "https://localhost:8443",
    secure: false
  }
}
```
A configuração acima garantirá que todas as solicitações para o caminho /v1 sejam roteadas para o serviço https://localhost:8443. Como o servidor de destino está ouvindo HTTPS, precisamos definir o secure como false se um certificado autoassinado for usado no servidor.

Para mais informação sobre configuração do proxy, por favor visitar o https://webpack.js.org/configuration/dev-server/
