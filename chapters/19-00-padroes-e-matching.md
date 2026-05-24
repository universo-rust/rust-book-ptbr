---
title: "Padrões e matching"
chapter_code: 19-00
slug: padroes-e-matching
---

# Padrões e Matching

Padrões são uma sintaxe especial em Rust para casar com a estrutura de tipos, tanto complexos quanto simples. Usar padrões em conjunto com expressões `match` e outras construções dá a você mais controle sobre o fluxo de controle de um programa. Um padrão consiste em alguma combinação dos seguintes elementos:

- Literais
- Arrays, enums, structs ou tuplas desestruturados
- Variáveis
- Curingas
- Placeholders

Alguns exemplos de padrões incluem `x`, `(a, 3)` e `Some(Color::Red)`. Nos contextos em que padrões são válidos, esses componentes descrevem o formato dos dados. Nosso programa então compara valores aos padrões para determinar se tem o formato correto de dados para continuar executando um trecho particular de código.

Para usar um padrão, comparamos ele com algum valor. Se o padrão casa com o valor, usamos as partes do valor no nosso código. Lembre das expressões `match` no Capítulo 6 que usavam padrões, como o exemplo da máquina de classificação de moedas. Se o valor encaixa no formato do padrão, podemos usar as partes nomeadas. Se não encaixa, o código associado ao padrão não será executado.

Este capítulo é uma referência sobre tudo relacionado a padrões. Cobriremos os lugares válidos para usar padrões, a diferença entre padrões refutáveis e irrefutáveis, e os diferentes tipos de sintaxe de padrão que você pode encontrar. Ao final do capítulo, você saberá como usar padrões para expressar muitos conceitos de forma clara.
