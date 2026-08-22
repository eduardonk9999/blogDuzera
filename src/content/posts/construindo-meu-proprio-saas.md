---
title: "Construindo meu próprio SaaS"
date: 2026-08-22
description: "Da ideia do hackathon até a modelagem de dados em Java com Spring Boot"
tags: ["saas", "java", "spring-boot", "arquitetura"]
draft: false
---

No meu último post, comentei que saí do hackathon da OpenAI com uma ideia na cabeça. Pois bem, resolvi levar essa ideia adiante.

## O começo: um protótipo em Node

Durante o hackathon, construí a primeira versão da API usando **Node.js** com a ajuda do Codex. A prioridade era velocidade: fazer algo funcionar para apresentar aos jurados.

E funcionou.

Mas quando decidi transformar aquilo em algo real, percebi que precisava repensar tudo desde o início.

## Por que refazer em Java com Spring Boot?

Pode parecer estranho jogar fora código que já funcionava. Mas a verdade é que aquele código foi feito para um hackathon, não para um produto.

Escolhi **Java com Spring Boot** por alguns motivos:

* É a stack que mais uso no dia a dia
* Tenho mais confiança para construir algo robusto
* O ecossistema para aplicações enterprise é muito maduro
* Quero aproveitar para aprofundar ainda mais meus conhecimentos

Além disso, dessa vez não estou com pressa. Posso fazer as coisas do jeito certo.

## Passando por todas as camadas

O que está sendo mais interessante nesse processo é perceber como construir um produto do zero te obriga a passar por **todas as camadas da engenharia de software**.

Não é só codar uma feature e fazer deploy.

É pensar em:

* Como os dados se relacionam
* Qual a estrutura das tabelas
* Quais entidades existem no domínio
* Como o sistema vai escalar
* Quais regras de negócio precisam existir

No trabalho, muitas vezes entramos em projetos que já têm tudo isso definido. Aqui, as decisões são todas minhas.

## Primeira etapa: modelagem de dados

Comecei pelo que considero a base de qualquer sistema: **a modelagem dos dados**.

Estou determinando:

* Quais tabelas o sistema vai ter
* O que cada tabela vai armazenar
* Como elas se relacionam entre si
* Quais campos são obrigatórios
* Quais constraints fazem sentido

Parece simples, mas cada decisão aqui impacta todo o resto. Uma modelagem mal feita agora vai gerar dor de cabeça lá na frente.

Por isso estou dedicando tempo para fazer isso com calma, pensando nos casos de uso e nas evoluções futuras do produto.

## O que vem pela frente

Depois da modelagem, vem a implementação das entidades, repositórios, services e controllers. A arquitetura clássica do Spring Boot.

Pretendo documentar esse processo aqui no blog, compartilhando o que aprendo e as decisões que tomo pelo caminho.

Ainda não sei se esse SaaS vai dar certo. Mas o aprendizado já está valendo.

E, no mínimo, vou sair disso com um projeto real no portfólio e muita experiência prática.
