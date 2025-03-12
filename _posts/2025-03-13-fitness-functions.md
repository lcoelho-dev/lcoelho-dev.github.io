---
title: Fitness Functions - O Check-up contínuo de seu Software
date: 2025-03-13 +/-TTTT
categories: [ architeture ]
tags: [ architeture, software ]
author: lcoelho
canonical_url: "https://lucascoelhosilva.github.io/posts/fitness-functions"
published: true
---

## Introdução

Para esta primeira publicação, quero começar com algo que venho lendo atualmente: o livro Arquitetura de Software: As Partes Difíceis. Esse livro me fez perceber que já aplicava alguns conceitos sem conhecê-los formalmente. Você já teve essa sensação ao ler algo e notar que já fazia aquilo intuitivamente? 😬

## O que são Fitness Functions? 👋

No capítulo 1 do livro, é mencionado o conceito de Fitness Functions, que foi popularizado pelo livro Building Evolutionary Architectures (Neal Ford, Rebecca Parsons e Patrick Kua).


> "Eles definiram o conceito de uma fitness function de uma arquitetura: qualquer mecanismo que realize uma avaliação objetiva da integridade de alguma característica da arquitetura ou a combinação de características da arquitetura".

Nada mais são do que mecanismos, automatizados ou não, que medem continuamente a qualidade de um aspecto arquitetural de um sistema. A ideia é definir métricas que garantam que a arquitetura evolua de forma controlada, sem degradar características essenciais. São feitas verificações contínuas para testar se um sistema ainda está "em forma" (fitness), em relação a critérios como performance, segurança, modularidade, disponibilidade, entre outros.


## Exemplos de Fitness Functions

As fitness functions podem ser divididas em categorias 

1. **Fitness Functions Module**
- Testes unitários para garantir que módulos em um projeto não criem dependências circulares.
- Análise de acoplamento entre serviços em uma arquitetura de microsserviços usando ferramentas como [ArchUnit](https://www.archunit.org) (Java).

2. **Fitness Functions Performance**
- Monitoramento para garantir que o tempo de resposta de uma API permaneça abaixo de um certo limite (ex: P95 < 500ms)
- Monitoramento para garantir que a latência de requisições externas não ultrapasse um limite definido 
- Monitoramento para garantir que a taxa de erros 5xx esteja abaixo de um limite aceitável

3. **Fitness Functions Code Quality**
- Regras no SonarQube para garantir que a cobertura de testes unitários nunca fique abaixo de um certo limite.
- Linters e analisadores de código estático para garantir padrões de nomenclatura e evitar complexidade excessiva.

4. **Fitness Functions Security**
- Testes de falha automatizados, como Chaos Engineering, para validar se um sistema continua funcionando quando um serviço crítico falha.
- Simulação de falhas na infraestrutura para garantir que um load balancer distribua corretamente o tráfego entre servidores.


## Como implementar fitness functions? 

Bom, depois dos exemplos acima, espero que você tenha tido ideias para incluí-las nos seus projetos (eu espero que sim hahaha).

Muitas vezes, fazemos isso de modo intrínseco e nem percebemos, como alertas que notificam o time quando a saúde da aplicação não está bem.

O ideal é identificar pontos críticos de uma aplicação e criar Fitness Functions para avaliá-los. Sempre tenha objetivos e métricas claras. Exemplo: O tempo de resposta do serviço Y deve ser menor que 300ms em 95% das requisições. A observabilidade do sistema é muito importante (isso é até um spoiler do próximo conteúdo 🫢).

As fitness functions podem ir além disso, garantindo que as camadas da aplicação sejam respeitadas. Isso é útil para novos membros da equipe que ainda não estão acostumados e precisam seguir os padrões. Alguns exemplos:

- DTOs sendo usados corretamente como response na controller, sem expor entidades ou domains.
- Evitar exceções genéricas no código.
- Seguir padrões arquiteturais estabelecidos.
- Garantir que a estrutura de pacotes siga um modelo pré-definido.
- Garantir que dependências de módulos sigam as regras estabelecidas.

Abaixo, um exemplo de código em Java usando ArchUnit para verificar se os tratamentos de erro estão sendo feitos corretamente, sem exceções genéricas, e se as camadas Controller, Usecases e Gateways estão sendo respeitadas:


```java
@AnalyzeClasses(packages = "com.coelho.api")
public class ModuleTest {

  @ArchTest
  private final ArchRule no_generic_exceptions = NO_CLASSES_SHOULD_THROW_GENERIC_EXCEPTIONS;

  @ArchTest
  private final ArchRule no_throw_generic_exceptions = NO_CLASSES_SHOULD_THROW_GENERIC_EXCEPTIONS;

  @ArchTest
  static final ArchRule layer_dependencies_are_respected = layeredArchitecture()
      .consideringAllDependencies()
      .layer("Controllers").definedBy("com.coelho.api.controllers..")
      .layer("Usecases").definedBy("com.coelho.api.usecases..")
      .layer("Gateways").definedBy("com.coelho.api.gateways.")

      .whereLayer("Controllers").mayNotBeAccessedByAnyLayer()
      .whereLayer("Usecases").mayOnlyBeAccessedByLayers("Controllers")
      .whereLayer("Gateways").mayOnlyBeAccessedByLayers("Usecases");
}
```

Espero que esta publicação sobre Fitness Functions tenha sido útil para você! Se você já usa algo parecido ou tem outras ideias, compartilhe nos comentários! 🚀
