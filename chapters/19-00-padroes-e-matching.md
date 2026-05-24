---
title: "Padrões e matching"
chapter_code: 19-00
slug: padroes-e-matching
---

# Padrões e Matching

Padrões são uma sintaxe especial em Rust para casar com a estrutura de tipos, simples ou complexos. Usar padrões com expressões `match` e outras construções dá mais controle ao fluxo do programa. Um padrão combina:

- Literais
- Arrays, enums, structs ou tuplas desestruturados
- Variáveis
- Curingas
- Placeholders

Exemplos: `x`, `(a, 3)`, `Some(Color::Red)`. Nos contextos em que padrões são válidos, esses componentes descrevem o formato dos dados. O programa compara valores aos padrões para saber se tem o formato certo para executar um trecho de código.

Para usar um padrão, comparamos com algum valor. Se o padrão casa, usamos as partes do valor no código. Lembre as expressões `match` do Capítulo 6, como o exemplo da máquina de moedas: se o valor encaixa no padrão, usamos as partes nomeadas; se não, o código daquele padrão não roda.

Este capítulo é referência sobre padrões: onde usá-los, diferença entre padrões refutáveis e irrefutáveis, e a sintaxe que você verá. Ao final, você saberá usar padrões para expressar muitos conceitos com clareza.
