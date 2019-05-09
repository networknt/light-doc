---
title: "Secret Encryption"
date: 2018-03-07T21:17:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

A plataforma Light tem um [decodificador] [] que pode ser injetado no aplicativo para descriptografar códigos criptografados em arquivos de configuração. Existe um utilitário separado que é usado para criptografar o secret.yml ou outros arquivos de configuração que contêm informações confidenciais.

Da perspectiva do consumidor, somente secret.yml será usado, a menos que o aplicativo cliente esteja tentando usar o
recurso para proteger outros arquivos de configuração específicos do cliente.

A seguir, um exemplo de secret.yml que pode ser usado junto com keystores client.yml e TLS.

```yaml
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: token1

# EmailSender password default address is noreply@lightapi.net
emailPassword: change-to-real-password

```


In most of the cases, the following fields are used by the client. 

```yaml

# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: token1

```

Para aprender como usar o encriptador e decodificador, por favor, consulte o [tutorial] [].

[decryptor]: /concern/decryptor/
[tutorial]: /tutorial/security/encrypt-decrypt/

