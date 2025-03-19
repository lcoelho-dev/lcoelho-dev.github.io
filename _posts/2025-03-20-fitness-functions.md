---
title: Fitness Functions - O Check-up contínuo de seu Software
date: 2025-03-20 08:00:00 +/-TTTT +/-TTTT
categories: [ code-quality ]
tags: [ software-quality, best-practices, fitness-functions, observability, java ]
author: lcoelho
canonical_url: "https://lcoelho-dev.github.io/posts/fitness-functions"
published: true
---

## Introdução

Para esta primeira publicação, quero começar com algo que venho lendo atualmente: o livro *Arquitetura de Software: As Partes Difíceis*. Esse livro me fez perceber que já aplicava alguns conceitos sem conhecê-los formalmente. Você já teve essa sensação ao ler algo e notar que já fazia aquilo intuitivamente? 😬

## O que são Fitness Functions? 👋

No capítulo 1 do livro, é mencionado o conceito de Fitness Functions, que foi popularizado pelo livro Building Evolutionary Architectures (Neal Ford, Rebecca Parsons e Patrick Kua).


**_trecho do livro_**
> "Eles definiram o conceito de uma fitness function de uma arquitetura: qualquer mecanismo que realize uma avaliação objetiva da integridade de alguma característica da arquitetura ou a combinação de características da arquitetura".


Nada mais são do que mecanismos, automatizados ou não, que medem continuamente a qualidade de um aspecto arquitetural de um sistema. A ideia é definir métricas que garantam que a arquitetura evolua de forma controlada, sem degradar características essenciais. São feitas verificações contínuas para testar se um sistema ainda está "em forma" (fitness), em relação a critérios como performance, segurança, modularidade, disponibilidade, entre outros.
Pensando nas funcionalidades core ou críticas do sistema.  


## Exemplos de Fitness Functions

Abaixo, listo algumas categorias onde Fitness Functions podem ser aplicadas, trazendo exemplos concretos para facilitar a compreensão e implementação.


![Code Quality](/assets/img/posts/fitness-functions/code-quality.png){: width="972" height="589" }

Porque fitness functions pode ajudar nesta etapa? Quais fitness functions podemos criar para garantir uma qualidade do software?

Durante o desenvolvimento, Fitness Functions podem ser usadas para garantir padrões de qualidade e facilitar a manutenção do código.
Criar Fitness Functions para garantir a qualidade do software pode trazer alguns benefícios para manutenibilidade do código, organizado e fácil de entender, código bem separado em módulos reutilizáveis e etc. e também ajuda a novos membros na equipe a seguir os padrões já definidos, garantindo por exemplo que as camadas da aplicação sejam respeitadas.

Pensando nas fitness functions para garantir a qualidade de código, podemos criar as seguintes:

## Fitness Functions Code Quality
- Cobertura de testes unitários nunca fique abaixo de um certo limite. [Jacoco](https://www.eclemma.org/jacoco)
- Linters e analisadores de código estático para garantir padrões de nomenclatura e evitar complexidade excessiva. [Google Java format](https://github.com/google/google-java-format)
- Testes unitários para garantir que módulos em um projeto não criem dependências circulares ou até mesmo não crie acoplamentos desnecessários [ArchUnit](https://www.archunit.org).

No livro ele menciona uma biblioteca em Java, o que achei bem interessante, realizar testes unitários para validar camadas ou até boas práticas de programação pode ajudar no processo de desenvolvimento e prevenir essas analises no processo de code review. (um passo a menos) 

Abaixo, um exemplo de código em Java usando para verificar se os tratamentos de erro estão sendo feitos corretamente, sem exceções genéricas, e se as camadas Controller, Usecases e Gateways estão sendo respeitadas:


```java
@AnalyzeClasses(packages = "com.coelho.api")
public class ModuleTest {

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

![Security](/assets/img/posts/fitness-functions/security.png){: width="972" height="589" }

Security é uma parte importante não podemos esquece-las e Fitness Functions com processos automatizados nos previnem de vulnerabilidades.

## Fitness Functions Security
- Vulnerabilidades em dependências. [OWASP](https://github.com/dependency-check/dependency-check-gradle)
- Exposição de dados sensíveis. [Gitleaks](https://github.com/gitleaks/gitleaks)
- Codigo com falhas de segurança. 


![Performance](/assets/img/posts/fitness-functions/performance.png){: width="972" height="589" }

A perfomance ou saúde do sistema tenho na minha visão que é o ponto chave se não tem algo funcionando alguém vai te chamar!! E não queremos isso haha
Ela pode ser analisada de várias maneiras, porém vou dar um foco para a ferramenta [Open Telemtry](https://opentelemetry.io/), ela nasceu no projeto Cloud Native Foundation totalmente Open Source e bem consolidada. Na real eles padronizaram as APIs para que seja analogo a implementação para que você seja livre de trocar as ferramentas onde analisa os dados, seja Jaeger ou zipkin, Loki ou Kibana, enfim... A observabilidade pode nos mostrar se a aplicação esta saúdavel ou não, temos muitas métricas para analisar isso porém vou deixar algumas para exemplificar: 

## Fitness Functions Performance 

[Open Telemtry](https://opentelemetry.io/)

- Monitoramento para garantir tempos de resposta aceitáveis. (ex: P95 < 500ms)
- Controle da taxa de erros 5xx.
- Monitoramento de CPU e memória.


```
increase(http_server_request_duration_seconds_count{http_server_request_duration_seconds_count=~"5.."}[5m]) >= 1
```

```
histogram_quantile(0.95, rate(http_server_request_duration_seconds_bucket[5m])) > 0.5
```

``` 
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
```


## Conclusão

Bom, depois dos exemplos acima, espero que você tenha tido ideias para incluí-las nos seus projetos (eu espero que sim hahaha). já utilizamos esses conceitos sem percebê-los formalmente, como alertas automáticos sobre problemas na saúde da aplicação.

O ideal é identificar pontos críticos de uma aplicação e criar Fitness Functions para avaliá-los. Sempre tenha objetivos e métricas claras. Exemplo: O tempo de resposta do serviço Y deve ser menor que 300ms em 95% das requisições. A observabilidade do sistema é muito importante (isso é até um spoiler do próximo conteúdo 🫢). Porém também é importante avaliar os casos, não queremos trazer mais complexidas no desenvolvimento e deixar algo ingesado por isso é bom ponderar.


Espero que esta publicação sobre Fitness Functions tenha sido útil para você! Se você já usa algo parecido ou tem outras ideias, compartilhe nos comentários! 🚀
