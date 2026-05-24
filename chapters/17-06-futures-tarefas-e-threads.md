---
title: "Juntando tudo: futures, tarefas e threads"
chapter_code: 17-06
slug: juntando-tudo-futures-tarefas-e-threads
---

# Juntando Tudo: Futures, Tarefas e Threads

Como vimos no Capítulo 16, threads são uma abordagem à concorrência. Neste capítulo vimos outra: async com futures e streams. Quando escolher uma ou outra? Depende — e muitas vezes não é threads _ou_ async, e sim _e_.

SOs oferecem threading há décadas; muitas linguagens suportam. Threads têm tradeoffs: memória por thread; em alguns embarcados não há SO nem threads.

Async oferece tradeoffs complementares. Operações concorrentes não precisam de thread própria; podem rodar em _tarefas_ (_tasks_), como com `trpl::spawn_task`. Tarefa parece thread, mas é gerenciada por código de biblioteca — o runtime.

Por isso `spawn` de thread e `spawn_task` são parecidos. Threads delimitam operações síncronas; concorrência _entre_ threads. Tarefas delimitam operações _assíncronas_; concorrência _entre_ e _dentro_ de tarefas, porque a tarefa alterna entre futures no corpo. Futures são a unidade mais fina; cada future pode ser árvore de outras. O executor do runtime gerencia tarefas; tarefas gerenciam futures. Tarefas são como threads leves do runtime, com capacidades extras.

Isso não torna tarefas sempre melhores que threads (nem o contrário). Threads têm modelo em alguns aspectos mais simples — pode ser força ou fraqueza. Threads são “dispara e esquece”; não há equivalente nativo a future; rodam até o SO interromper.

Threads e tarefas combinam bem: tarefas podem migrar entre threads (work stealing no runtime que usamos, incluindo `spawn_blocking` e `spawn_task`). Muitos runtimes são multithread por padrão.

Regras práticas:

- Trabalho _muito paralelizável_ (limitado por CPU), como processar dados em partes independentes: threads costumam ser melhores.
- Trabalho _muito concorrente_ (limitado por E/S), como muitas fontes de mensagens em intervalos diferentes: async costuma ser melhor.

Precisando dos dois, combine livremente. A Listagem 17-25 é um padrão comum.

**Arquivo: src/main.rs**

```rust
use std::{thread, time::Duration};

fn main() {
    let (tx, mut rx) = trpl::channel();

    thread::spawn(move || {
        for i in 1..11 {
            tx.send(i).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    trpl::block_on(async {
        while let Some(message) = rx.recv().await {
            println!("{message}");
        }
    });
}
```

<a id="listagem-17-25"></a>

[Listagem 17-25](#listagem-17-25): Enviar com código bloqueante numa thread e aguardar mensagens num bloco async

Channel async, thread com `move` no sender enviando 1..10 com `sleep` de um segundo, e `block_on` com bloco async que aguarda as mensagens.

Volte ao cenário do início do capítulo: encoding de vídeo em thread dedicada (CPU-bound) e notificar a UI por channel async. Há inúmeros exemplos assim na prática.

## Resumo

Não é o último contato com concorrência neste livro. O projeto do Capítulo 21 aplica esses conceitos de forma mais realista e compara threads com tarefas e futures.

Seja qual for a abordagem, o Rust oferece ferramentas para código concorrente seguro e rápido — de servidor web de alto throughput a SO embarcado.

A seguir, formas idiomáticas de modelar problemas e estruturar soluções conforme os programas crescem, e como os idiomas do Rust se relacionam com programação orientada a objetos.
