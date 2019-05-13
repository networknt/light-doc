---
title: "TLS Connection"
date: 2018-03-01T21:36:44-05:00
description: ""
categories: []
keywords: []
weight: 20
aliases: []
toc: false
draft: false
---

A plataforma light suporta SSL de uma via por padrão no [light-codegen] [] e SSL de duas vias atualizando server.yml para ativar. A menos que você esteja usando algumas ferramentas antigas que não suportam HTTPS, recomenda-se usar pelo menos SSL unidirecional, mesmo na fase de desenvolvimento, para que você não tenha nenhuma surpresa ao liberar para um ambiente de teste oficial.

### certificados TLS

Existem quatro arquivos keystore que podem ser gerados a partir do light-codegen, dependendo do config.json no repositório model-config.

Aqui está um exemplo de config.json para light-codegen.

```
{
  "name": "apia",
  "version": "1.0.0",
  "groupId": "com.networknt",
  "artifactId": "apia",
  "rootPackage": "com.networknt.apia",
  "handlerPackage":"com.networknt.apia.handler",
  "modelPackage":"com.networknt.apia.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 7001,
  "enableHttp": true,
  "httpsPort": 7441,
  "enableHttps": true,
  "enableRegistry": false,
  "supportDb": false,
  "supportH2ForTest": false,
  "supportClient": true
}

```
Por padrão, o código gerado terá server.keystore e server.truststore na pasta de configuração. Mas se supportClient for true em config.json, o client.keystore e o client.truststore também serão gerados.

Para obter informações sobre arquivos de armazenamento de chaves, consulte [armazenamento confiável de keystore][].

Os keystores e truststores gerados contêm certificados auto-assinados expiram no ano de 2023 e devem ser usados apenas para desenvolvimento. Depois de passar para um ambiente de teste oficial, eles precisam ser substituídos por outros certificados auto-assinados ou certificados assinados pela CA.

Por favor consulte o [certificado assinado por si mesmo vs assinado pela CA] [] para obter detalhes sobre quando usar o certificado autoassinado ou assinado pela CA.

### Habilitar TLS direcional ou bidirecional

Por favor consulte [server config][] para mais detalhes.

### Download certificado do servidor

Ao se conectar a um servidor com HTTPS, você deve solicitar o certificado de cliente do administrador do servidor. Se você não puder obter o certificado do administrador do servidor, poderá baixá-lo do servidor com o openssl.

Por favor, consulte o tutorial [public https][] para mais detalhes.

### Conexão de TLS modo Debug 

Ao fazer conexão TLS com o servidor, você precisa adicionar certificados ao client.truststore na maioria dos casos. Para a maioria dos desenvolvedores, pode ser um desafio fazer tudo certo na primeira vez. Se a conexão não for estabelecida com o servidor, é provável que você tenha o client.truststore sem o certificado do cliente. Para descobrir se o problema de conexão é devido ao certificado, você pode habilitar a depuração do tls em seu IDE.

Aqui está um artigo que contém todos os detalhes em [Debugging SSL / TLS Connections][].

Basicamente, você precisa colocar a seguinte opção JVM ao iniciar seu servidor no IDE.

```
-Djavax.net.debug=all
```

[keystore truststore]: /tutorial/security/keystore-truststore/
[self-signed vs CA-signed certificate]: /faq/self-ca-signed-cert/
[server config]: /concern/server/
[light-codegen]: /tool/light-codegen/
[public https]: /tutorial/client/public-https/
[Debugging SSL/TLS Connections]: https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html

