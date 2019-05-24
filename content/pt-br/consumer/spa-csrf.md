---
title: "SPA CSRF Prevention"
date: 2018-05-22T12:48:01-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Cross-Site Request Forgery (CSRF) é um tipo de ataque que ocorre quando um site, email, blog, mensagem instantânea ou programa mal-intencionado faz com que o navegador da Web de um usuário execute uma ação indesejada em um site confiável para o qual o usuário está atualmente autenticado. O impacto de um ataque CSRF bem-sucedido é limitado aos recursos expostos pelo aplicativo vulnerável. Por exemplo, esse ataque pode resultar em uma transferência de fundos, alteração de senha ou compra de um item no contexto do usuário. Com efeito, ataques de CSRF são usados por um invasor para fazer com que um sistema de destino execute uma função através do navegador de destino sem o conhecimento do usuário de destino, pelo menos até que a transação não autorizada seja confirmada.

Os impactos das explorações bem-sucedidas do CSRF variam muito com base nos privilégios de cada vítima. Ao segmentar um usuário normal, um ataque CSRF bem-sucedido pode comprometer os dados do usuário final e suas funções associadas. Se o usuário final direcionado for uma conta de administrador, um ataque de CSRF pode comprometer todo o aplicativo da web. Os sites com maior probabilidade de serem atacados pelo CSRF são sites da comunidade (redes sociais, email) ou sites que possuem contas de alto valor em dólares associados a eles (bancos, corretoras de valores, serviços de pagamento de contas). Utilizando a engenharia social, um invasor pode incorporar códigos maliciosos de HTML ou JavaScript em um email ou site para solicitar uma "URL de tarefa" específica. A tarefa, em seguida, executa com ou sem o conhecimento do usuário, diretamente ou utilizando uma falha Cross-Site Scripting (ex: Samy MySpace Worm).
