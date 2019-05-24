---
title: "HTTP 2.0"
date: 2018-03-01T21:37:12-05:00
description: ""
categories: []
keywords: []
weight: 30
aliases: []
toc: false
draft: false
---

Por padrão, os serviços construídos em cima da plataforma light têm o HTTP 2.0 ativado automaticamente
no light-codegen. O servidor também suporta conexão HTTP 1.1 se o cliente não puder
negociar o HTTP 2.0 com o servidor.

Existem muitos benefícios com o HTTP 2.0, mas o JDK 8 ou menor não suportam nativamente. Apesar
que existem algumas alternativas, nenhuma delas é uma solução prática. Dada a situação, temos
construído módulo cliente baseado no núcleo Undertow para suporte a HTTP 2.0.

Segue alguns links a respeito de HTTP 2.0 e HTTP 1.1

https://www.pacwebhosting.co.uk/insight/News/NewsItem/View/31/http11-vs-http2-whats-the-difference

https://imagekit.io/demo/http2-vs-http1

https://www.upwork.com/hiring/development/the-http2-protocol-its-pros-cons-and-how-to-start-using-it/

### Configuração do servidor

Para habilitar HTTP 2.0 no lado do servidor, você precisa garantir que enableHttp2 possui o valor true no 
arquivo de configuração server.yml.

Por favor referencie ao [server module] e configuração do server.yml para mais informações.

### Módulo cliente

Ao usar o módulo do cliente light-4j para fazer uma solicitação HTTP para o servidor, o HTTP 2.0 não está ativado
por padrão. Você precisa especificar explicitamente que deseja estabelecer uma conexão HTTP 2.0.

Aqui está o exemplo de código para criar essa conexão:
```java
        if(connection == null || !connection.isOpen()) {
            try {
                connection = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
```

A opção abaixo criará uma conexão HTTP 2.0
```java

OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)

```

Se você tem certeza de que o serviço é construído em light-4j e o HTTP 2.0 não está desabilitado. Então você deveria
usar o HTTP 2.0 do lado do consumidor para aproveitar a velocidade rápida e multiplexação de conexão única.

### Outro HTTP cliente java

A maioria dos clientes HTTP Java não suporta HTTP 2.0 ou você tem que encontrar uma solução alternativa para fazer isso com
bootclasspath jar file por versão do JDK em Java 8. Se você estiver usando o Java 9, então ele tem
suporte para HTTP 2.0; no entanto, nunca foi testado com o framework básico.

[server module]: /concern/server/
