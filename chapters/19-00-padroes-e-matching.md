---
title: "Padrões e matching"
chapter_code: 19-00
slug: padroes-e-matching
---

# Padrões e matching

Padrões são uma sintaxe especial em Rust para casar com a estrutura de tipos, sejam eles complexos ou simples. Usar padrões junto com expressões `match` e outras construções dá mais controle sobre o fluxo de controle de um programa. Um padrão é formado por alguma combinação dos seguintes elementos:

- Literais
- Arrays, enums, structs ou tuplas desestruturados
- Variáveis
- Curingas
- Placeholders

Alguns exemplos de padrões são `x`, `(a, 3)` e `Some(Color::Red)`. Nos contextos em que padrões são válidos, esses componentes descrevem o formato dos dados. Nosso programa então compara valores com esses padrões para decidir se tem o formato de dados correto para continuar executando um determinado trecho de código.

Para usar um padrão, comparamos esse padrão com algum valor. Se o padrão casa com o valor, usamos as partes do valor no nosso código. Lembre-se das expressões `match` do Capítulo 6 que usavam padrões, como o exemplo da máquina de separação de moedas. Se o valor se encaixa no formato do padrão, podemos usar as partes nomeadas. Se não se encaixa, o código associado ao padrão não é executado.

Este capítulo é uma referência sobre tudo que envolve padrões. Vamos cobrir os lugares em que padrões podem ser usados, a diferença entre padrões refutáveis e irrefutáveis e os diferentes tipos de sintaxe de padrão que você pode encontrar. Ao fim do capítulo, você saberá usar padrões para expressar muitos conceitos de forma clara.
