---
title: "Desempenho em loops vs. iterators"
chapter_code: 13-04
slug: desempenho-em-loops-vs-iterators
---

# Desempenho em Loops vs. Iterators

Para decidir se deve usar loops ou iterators, você precisa saber qual implementação é mais rápida: a versão da função `search` com um loop `for` explícito ou a versão com iterators.

Executamos um benchmark carregando o conteúdo inteiro de _The Adventures of Sherlock Holmes_, de Sir Arthur Conan Doyle, em um `String` e buscando a palavra _the_ no conteúdo. Estes são os resultados do benchmark na versão de `search` que usa o loop `for` e na versão que usa iterators:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

As duas implementações têm desempenho semelhante! Não explicaremos o código do benchmark aqui porque o ponto não é provar que as duas versões são equivalentes, mas ter uma noção geral de como essas duas implementações se comparam em termos de desempenho.

Para um benchmark mais abrangente, você deveria testar com vários textos de vários tamanhos como `contents`, palavras diferentes e palavras de comprimentos diferentes como `query`, e todo tipo de outras variações. O ponto é este: iterators, embora sejam uma abstração de alto nível, compilam para código praticamente igual ao que você escreveria manualmente em nível mais baixo. Iterators são uma das _abstrações de custo zero_ do Rust, o que significa que usar a abstração não impõe overhead adicional em tempo de execução. Isso é análogo a como Bjarne Stroustrup, o designer e implementador original de C++, define _zero-overhead_ em sua apresentação de keynote ETAPS 2012 “Foundations of C++”:

> Em geral, implementações de C++ obedecem ao princípio de zero-overhead: o que você não usa, você não paga. E além disso: o que você usa, você não conseguiria codificar à mão melhor.

Em muitos casos, código Rust que usa iterators compila para o mesmo assembly que você escreveria à mão. Otimizações como desenrolamento de loop e eliminação de verificação de limites em acesso a arrays se aplicam e tornam o código resultante extremamente eficiente. Agora que você sabe disso, pode usar iterators e closures sem medo! Eles fazem o código parecer de nível mais alto, mas não impõem penalidade de desempenho em tempo de execução por isso.

## Resumo

Closures e iterators são recursos do Rust inspirados em ideias de linguagens de programação funcional. Eles contribuem para a capacidade do Rust de expressar ideias de alto nível com desempenho de baixo nível. As implementações de closures e iterators são tais que o desempenho em tempo de execução não é afetado. Isso faz parte do objetivo do Rust de oferecer abstrações de custo zero.

Agora que melhoramos a expressividade do nosso projeto de I/O, vamos olhar mais recursos do `cargo` que nos ajudarão a compartilhar o projeto com o mundo.
