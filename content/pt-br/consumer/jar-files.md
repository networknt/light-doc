---
title: "Arquivos Jar"
date: 2018-05-07T12:30:59-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Ao usar o módulo cliente a partir de qualquer aplicativo Java construído sobre o Java 8, mas não na plataforma light, vários módulos precisam ser incluídos juntos para executar várias funções, incluindo solicitação HTTP/2, descoberta de serviço, balanceamento de carga e suporte a cluster. Esses arquivos jar são pequenos e contêm apenas de 2 a 5 classes. A lista a seguir inclui todos os arquivos jar relacionados do Maven Central ao usar o módulo do cliente.


Para todos os módulos listados abaixo, há 3 arquivos de configurações que precisa ser externalizado para funcionar, são:

1. client.yml - Define configuração do cliente e comunicação OAuth 2.0. É apenas utilizado pelo módulo cliente.
2. service.yml - balance, cluster, registry, consul e configuração de decifradora são definidas nesse arquivo. 
3. secret.yml - OAuth 2.0 client secret e senha de certificado etc. são definidos neste arquivo para facilitar o gerenciamento de segredos. Todos os secredos podem ser encriptados em texto não claros para ser visto no arquivo de 
configuração. 

* [client.jar][]

É o arquivo jar principal que é responsável por criar uma conexão HTTP/2, enviar solicitações e invocar chamadas de retorno do cliente. Ele também lida com a renovação automática de tokens JWT do provedor do OAuth 2.0 e passa pelo correlationId e traceabilityId.

* [config.jar][]

Ele carrega arquivos de configuração dentro do módulo, aplicativo ou diretório externalizado.

* [balance.jar][]

É o módulo para lidar com o balanceamento de carga do lado do cliente. Suporta três estratégias integradas: Round Robin, Local First e Consistent Hashing.

* [cluster.jar][]

Suporte de cluster no lado do cliente, ele usa o módulo de balanceamento para pegar um candidato (IP e porta) de uma lista de instâncias descobertas.

* [registry.jar][]

Ele fornece lógica de descoberta de serviço geral junto com implementações concretas como Consul ou Zookeeper.

* [consul.jar][]

É um módulo cliente para a comunicação do agente Consul.

* [service.jar][]

Serviço de injeção de dependência que é responsável por conectar todos os outros módulos.

* [decryptor.jar][]

É um módulo opcional se você quiser criptografar dados confidenciais dos arquivos de configuração.

* [common.jar][]

É um módulo opcional e usado pelo decifrador pelo DecryptUtil.java.

[client.jar]: /concern/client/
[config.jar]: /concern/config/
[balance.jar]: /concern/balance/
[cluster.jar]: /concern/cluster/
[registry.jar]: /concern/registry/
[consul.jar]: /concern/consul/
[service.jar]: /concern/service/
[decryptor.jar]: /concern/decryptor/
[common.jar]: /concern/common/

