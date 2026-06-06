---
title: "Recursos funcionais: iterators e closures"
chapter_code: 13-00
slug: recursos-funcionais-iterators-e-closures
challenge_day: 17
reading_minutes: 2
---

# Recursos de linguagens funcionais: iterators e closures

O design do Rust se inspirou em muitas linguagens e técnicas existentes, e uma influência significativa é a _programação funcional_. Programar em estilo funcional costuma incluir usar funções como valores, passando-as como argumentos, retornando-as de outras funções, atribuindo-as a variáveis para execução posterior e assim por diante.

Neste capítulo, não vamos debater o que é ou não é programação funcional; em vez disso, discutiremos alguns recursos do Rust semelhantes a recursos de muitas linguagens frequentemente chamadas de funcionais.

Mais especificamente, cobriremos:

- _Closures_, uma construção semelhante a funções que você pode armazenar em uma variável
- _Iterators_, uma forma de processar uma série de elementos
- Como usar closures e iterators para melhorar o projeto de I/O do Capítulo 12
- O desempenho de closures e iterators (spoiler: são mais rápidos do que você imagina!)

Já cobrimos outros recursos do Rust, como pattern matching e enums, que também são influenciados pelo estilo funcional. Como dominar closures e iterators é parte importante de escrever código Rust rápido e idiomático, dedicaremos este capítulo inteiro a eles.
