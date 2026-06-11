---
title: "Juntando tudo: futures, tarefas e threads"
chapter_code: 17-06
slug: juntando-tudo-futures-tarefas-e-threads
---

# Juntando tudo: futures, tarefas e threads

Como vimos no [Capítulo 16](/livro/cap16-00-concorrencia-sem-medo), threads oferecem uma abordagem para concorrência. Neste capítulo, vimos outra: usar async com futures e streams. Se você está se perguntando quando escolher um método ou outro, a resposta é: depende! E, em muitos casos, a escolha não é entre threads _ou_ async, mas entre threads _e_ async trabalhando juntos.

Há décadas, muitos sistemas operacionais oferecem modelos de concorrência baseados em threads, e muitas linguagens de programação dão suporte a eles por consequência. Esses modelos, porém, têm seus custos. Em muitos sistemas operacionais, cada thread usa uma quantidade considerável de memória. Threads também só são uma opção quando o sistema operacional e o hardware dão suporte a elas. Diferentemente de computadores desktop e dispositivos móveis comuns, alguns sistemas embarcados nem sequer têm um sistema operacional; portanto, também não têm threads.

O modelo async oferece um conjunto diferente, e no fim complementar, de tradeoffs. No modelo async, operações concorrentes não precisam ter suas próprias threads. Em vez disso, elas podem rodar em tarefas, como quando usamos `trpl::spawn_task` para iniciar trabalho a partir de uma função síncrona na seção sobre streams. Uma tarefa é parecida com uma thread, mas, em vez de ser gerenciada pelo sistema operacional, é gerenciada por código em nível de biblioteca: o runtime.

Há uma razão para as APIs de criação de threads e de tarefas serem tão parecidas. Threads funcionam como uma fronteira para conjuntos de operações síncronas; a concorrência é possível _entre_ threads. Tarefas funcionam como uma fronteira para conjuntos de operações _assíncronas_; a concorrência é possível tanto _entre_ tarefas quanto _dentro_ delas, porque uma tarefa pode alternar entre as futures em seu corpo. Por fim, futures são a unidade mais granular de concorrência em Rust, e cada future pode representar uma árvore de outras futures. O runtime, mais especificamente seu executor, gerencia tarefas; as tarefas gerenciam futures. Nesse sentido, tarefas são parecidas com threads leves gerenciadas pelo runtime, com capacidades extras por serem controladas pelo runtime em vez do sistema operacional.

Isso não significa que tarefas async sejam sempre melhores que threads, nem o contrário. Concorrência com threads é, em alguns aspectos, um modelo de programação mais simples que concorrência com `async`. Isso pode ser uma vantagem ou uma desvantagem. Threads são um pouco do tipo "dispare e esqueça": elas não têm um equivalente nativo a uma future, então simplesmente rodam até terminar, sem serem interrompidas exceto pelo próprio sistema operacional.

Na prática, threads e tarefas muitas vezes funcionam muito bem juntas, porque tarefas podem, pelo menos em alguns runtimes, ser movidas entre threads. Na verdade, por baixo dos panos, o runtime que usamos neste capítulo, incluindo as funções `spawn_blocking` e `spawn_task`, é multithread por padrão! Muitos runtimes usam uma abordagem chamada _work stealing_ para mover tarefas de forma transparente entre threads, de acordo com a utilização atual de cada thread, melhorando o desempenho geral do sistema. Essa abordagem, na verdade, exige threads _e_ tarefas e, portanto, futures.

Ao pensar em qual método usar em cada situação, considere estas regras práticas:

- Se o trabalho é _muito paralelizável_, isto é, limitado por CPU, como processar muitos dados em que cada parte pode ser tratada separadamente, threads costumam ser a melhor escolha.
- Se o trabalho é _muito concorrente_, isto é, limitado por E/S, como lidar com mensagens vindas de várias fontes em intervalos ou ritmos diferentes, async costuma ser a melhor escolha.

E, se você precisa tanto de paralelismo quanto de concorrência, não precisa escolher entre threads e async. Você pode combiná-los livremente, deixando cada um fazer o papel em que é melhor. A Listagem 17-25 mostra um exemplo bastante comum desse tipo de mistura em código Rust real.

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

[Listagem 17-25](#listagem-17-25): Enviando mensagens com código bloqueante em uma thread e aguardando as mensagens em um bloco async

Começamos criando um channel async. Em seguida, criamos uma thread que assume ownership do lado de envio do channel usando a palavra-chave `move`. Dentro da thread, enviamos os números de 1 a 10, com `sleep` de um segundo entre cada envio. Por fim, executamos uma future criada com um bloco async passado para `trpl::block_on`, como fizemos ao longo de todo o capítulo. Nessa future, aguardamos as mensagens, assim como nos outros exemplos de passagem de mensagens que vimos.

Voltando ao cenário com que abrimos o capítulo, imagine executar um conjunto de tarefas de codificação de vídeo em uma thread dedicada, porque codificação de vídeo é limitada por CPU, mas notificar a interface de usuário de que essas operações terminaram usando um channel async. Há incontáveis exemplos desse tipo de combinação em casos de uso reais.

## Resumo

Este não será seu último contato com concorrência neste livro. O projeto do [Capítulo 21](/livro/cap21-00-projeto-final-um-servidor-web) aplicará esses conceitos em uma situação mais realista do que os exemplos simples discutidos aqui e comparará de forma mais direta a resolução de problemas com threads versus tarefas e futures.

Seja qual for a abordagem escolhida, Rust oferece as ferramentas necessárias para escrever código concorrente seguro e rápido, seja para um servidor web de alta vazão, seja para um sistema operacional embarcado.

A seguir, falaremos sobre formas idiomáticas de modelar problemas e estruturar soluções à medida que seus programas Rust ficam maiores. Além disso, discutiremos como os idioms de Rust se relacionam com aqueles que talvez você conheça da programação orientada a objetos.
