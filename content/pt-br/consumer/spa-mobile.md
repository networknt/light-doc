---
title: "SPA and Mobile Discovery"
date: 2018-04-30T19:58:40-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Muitas vezes, quando discutimos a descoberta de serviços, aplicativos e serviços autônomos são mencionados como clientes. Raramente discutimos o SPA em execução no navegador e o aplicativo móvel em execução nos dispositivos, pois eles não são considerados para executar a descoberta de serviços.

Na maioria dos casos, esses aplicativos se conectam da Internet a um balanceador de carga estática como o F5 e, em seguida, fazem o roteamento para várias instâncias BFF(Backend for Frontend) endereçadas estáticas. Portanto, mesmo que seja capaz de realizar descobertas a partir do SPA ou do aplicativo móvel, não há necessidade de fazê-lo, pois as instâncias do BFF são acessadas diretamente com endereços IP. Além disso, BFF é responsável por descobrir microsserviços em execução na nuvem com endereços IP dinâmicos e números de porta.

A razão pela qual temos esse tipo de arquitetura é abordar as preocupações de segurança. Nossos serviços internos em execução na nuvem podem se conectar a bancos de dados, repositórios ou outros sistemas de back-end importantes. Eles não devem ser acessados diretamente da Internet. Eles residem na rede interna e só podem ser acessados através de BFFs que estão sendo executados no DMZ.

Atualmente, o design típico seria um gateway de API que atua como um BFF, que lida com todas as solicitações externas e executa a filtragem de segurança. Ele funciona com serviços da Web monolíticos, mas não para microsserviços, pois o único gateway de API é o gargalo e o único ponto de falha.

Ao adotar a arquitetura de microsserviços, precisamos de gateways de API distribuídos para que a carga de trabalho possa ser balanceada para diferentes instâncias especificamente projetadas para cada cliente. Esses gateways distribuídos conhecem as solicitações do cliente muito melhor e lidam com a segurança do cliente com o provedor do OAuth 2.0 para proteger a lógica complexa dos desenvolvedores clientes. Idealmente, esses gateways distribuídos devem ser construídos como microsserviços para garantir a escalabilidade a longo prazo.

Na plataforma leve, três componentes podem atuar como um gateway distribuído, conhecido como BFF.


### Servidor web existente

Se suas APIs foram criadas sobre o servidor da Web existente e não podem migrar para a arquitetura de microsserviços durante a noite, você ainda poderá usar seu servidor da Web como um gateway para a descoberta de serviços. Isso requer que o servidor da Web seja construído no Java 8 e superior, de modo que o módulo do cliente light-4j possa ser utilizado para interagir com o provedor do OAuth 2.0 e com o Consul para descoberta de serviço. Se o servidor da Web não puder usar o módulo do cliente incorporado, o light-router poderá ser usado. Nesse design, o servidor da Web também atua como um agregador, de modo que uma solicitação do cliente possa invocar vários microsserviços no backend.

### Agregador customizado

Para novos aplicativos clientes, você pode construir um agregador no framework light-rest-4j ou light-graphql-4j como um microsserviço. Ele pode simplesmente simplificar a lógica de chamada do cliente e tornar os desenvolvedores do lado do cliente mais confortáveis. Essa camada também oculta os detalhes dos serviços de back-end para que nossas APIs de serviços de back-end não sejam expostas diretamente à Internet.

### Roteador Light

Com o HTTP 2.0 se popularizando, as necessidades de agregador não são tão importantes, já que um cliente pode acessar vários serviços simultaneamente sem penalidade de desempenho.

Essa arquitetura também abre a porta para microsserviços verdadeiros, o que significa que a interface do usuário e a API são totalmente combinadas como um serviço que lida com uma função de negócios específica. Com estruturas do lado do cliente, como React ou React Native, a limitação é a sua imaginação.
