---
title: "Streams: futures em sequência"
chapter_code: 17-04
slug: streams-futures-em-sequencia
---

# Streams: Futures em Sequência

Lembre o receptor do channel async em Enviando dados entre duas tarefas com passagem de mensagens: `recv` produz uma sequência de itens ao longo do tempo. Isso é um _stream_ — padrão mais geral: fila, chunks de arquivo grandes demais para RAM, dados de rede chegando aos poucos. Streams são futures; combinam-se com outras futures (lotes de eventos, timeout em operações longas, throttle na UI).

Vimos sequência de itens no Capítulo 13 em A trait Iterator e o método `next`, mas há duas diferenças em relação ao receptor async: tempo (iterator síncrono, channel assíncrono) e API (`Iterator::next` versus `recv` assíncrono). A semelhança não é acidente: stream é iteração assíncrona; `Receiver` espera mensagens, a API geral de stream oferece o próximo item como `Iterator`, de forma assíncrona.

Podemos criar stream a partir de iterator e chamar `next` com await (Listagem 17-21) — que ainda não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
        let values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
        let iter = values.iter().map(|n| n * 2);
        let mut stream = trpl::stream_from_iter(iter);

        while let Some(value) = stream.next().await {
            println!("The value was: {value}");
        }
```

[Listagem 17-21](#listagem-17-21): Criando stream a partir de iterator e imprimindo valores

Erro típico:

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

Precisamos da trait certa no escopo. Poderíamos esperar `Stream`, mas é `StreamExt` (_extension_ — padrão comum em Rust).

`Stream` combina `Iterator` e `Future` em interface de baixo nível. `StreamExt` oferece API de alto nível, incluindo `next` e utilitários como em `Iterator`. Ainda não estão na biblioteca padrão; a maioria dos crates do ecossistema usa definições parecidas.

Correção: `use trpl::StreamExt` (Listagem 17-22).

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

[Listagem 17-22](#listagem-17-22): Usando iterator como base de stream com sucesso

Com `StreamExt` no escopo, usamos os métodos utilitários como com iterators.
