---
title: "Streams: futures em sequência"
chapter_code: 17-04
slug: streams-futures-em-sequencia
challenge_day: 23
reading_minutes: 4
---

# Streams: futures em sequência

Lembre-se de como usamos o receptor do nosso channel async na seção ["Enviando dados entre duas tarefas com passagem de mensagens"](/livro/cap17-02-aplicando-concorrencia-com-async#enviando-dados-entre-duas-tarefas-com-passagem-de-mensagens). O método async `recv` produz uma sequência de itens ao longo do tempo. Esse é um caso de um padrão bem mais geral conhecido como _stream_. Muitas ideias são naturalmente representadas como streams: itens ficando disponíveis em uma fila, pedaços de dados sendo lidos aos poucos do sistema de arquivos quando o conjunto completo é grande demais para caber na memória, ou dados chegando pela rede ao longo do tempo. Como streams são futures, podemos usá-las com outros tipos de futures e combiná-las de formas interessantes. Por exemplo, podemos agrupar eventos para evitar chamadas de rede em excesso, aplicar timeouts a sequências de operações demoradas ou limitar a frequência de eventos de interface para evitar trabalho desnecessário.

Já vimos uma sequência de itens no Capítulo 13, quando estudamos a trait `Iterator` na seção ["A trait `Iterator` e o método `next`"](/livro/cap13-02-processando-uma-serie-de-itens-com-iterators#a-trait-iterator-e-o-metodo-next), mas há duas diferenças entre iterators e o receptor de um channel async. A primeira diferença é o tempo: iterators são síncronos, enquanto o receptor do channel é assíncrono. A segunda diferença é a API. Ao trabalhar diretamente com `Iterator`, chamamos o método síncrono `next`. No caso específico do stream `trpl::Receiver`, chamamos um método assíncrono chamado `recv`. Fora isso, as APIs parecem bastante parecidas, e essa semelhança não é coincidência. Um stream é como uma forma assíncrona de iteração. Enquanto `trpl::Receiver` espera especificamente por mensagens, a API geral de streams é mais ampla: ela fornece o próximo item como `Iterator` faz, mas de maneira assíncrona.

A semelhança entre iterators e streams em Rust significa que podemos criar um stream a partir de qualquer iterator. Assim como fazemos com um iterator, podemos trabalhar com um stream chamando seu método `next` e aguardando a saída, como na Listagem 17-21, que ainda não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let iter = values.iter().map(|n| n * 2);
let mut stream = trpl::stream_from_iter(iter);

while let Some(value) = stream.next().await {
    println!("The value was: {value}");
}
```

<a id="listagem-17-21"></a>

[Listagem 17-21](#listagem-17-21): Criando um stream a partir de um iterator e imprimindo seus valores

Começamos com um array de números, convertemos esse array em um iterator e chamamos `map` para dobrar todos os valores. Depois, convertemos o iterator em um stream usando a função `trpl::stream_from_iter`. Em seguida, percorremos os itens do stream conforme eles chegam usando um loop `while let`.

Infelizmente, ao tentar executar esse código, ele não compila. Em vez disso, o compilador informa que não há nenhum método `next` disponível:

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Como essa saída explica, o erro acontece porque precisamos colocar a trait correta em escopo para poder usar o método `next`. Pelo que discutimos até aqui, seria razoável esperar que essa trait fosse `Stream`, mas na verdade ela é `StreamExt`. Abreviação de _extension_, `Ext` é um padrão comum na comunidade Rust para estender uma trait com outra.

A trait `Stream` define uma interface de baixo nível que, na prática, combina as traits `Iterator` e `Future`. `StreamExt` oferece um conjunto de APIs de nível mais alto sobre `Stream`, incluindo o método `next` e outros métodos utilitários parecidos com os fornecidos por `Iterator`. `Stream` e `StreamExt` ainda não fazem parte da biblioteca padrão de Rust, mas a maioria dos crates do ecossistema usa definições parecidas.

A correção para o erro do compilador é adicionar uma instrução `use` para `trpl::StreamExt`, como na Listagem 17-22.

**Arquivo: src/main.rs**

```rust
use trpl::StreamExt;

fn main() {
    trpl::block_on(async {
        let values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
        let iter = values.iter().map(|n| n * 2);
        let mut stream = trpl::stream_from_iter(iter);

        while let Some(value) = stream.next().await {
            println!("The value was: {value}");
        }
    });
}
```

<a id="listagem-17-22"></a>

[Listagem 17-22](#listagem-17-22): Usando um iterator como base para um stream com sucesso

Com todas essas peças juntas, o código funciona como queremos. Além disso, agora que `StreamExt` está em escopo, podemos usar todos os seus métodos utilitários, assim como fazemos com iterators.
