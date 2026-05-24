---
title: "Aplicando concorrência com async"
chapter_code: 17-02
slug: aplicando-concorrencia-com-async
---

# Aplicando Concorrência com Async

Nesta seção, aplicamos async a alguns dos mesmos desafios de concorrência que enfrentamos com threads no Capítulo 16. Como já discutimos muitas ideias lá, focamos no que muda entre threads e futures.

Em muitos casos, as APIs de concorrência com async são muito parecidas com as de threads. Em outros, bem diferentes. Mesmo quando _parecem_ iguais, o comportamento e o desempenho costumam divergir.

## Criando uma nova tarefa com `spawn_task`

A primeira operação em Criando uma nova thread com `spawn` foi contar em duas threads. Faremos o mesmo com async. O crate `trpl` oferece `spawn_task`, parecido com `thread::spawn`, e `sleep`, versão async de `thread::sleep` (Listagem 17-6).

**Arquivo: src/main.rs**

```rust
use std::time::Duration;

fn main() {
    trpl::block_on(async {
        trpl::spawn_task(async {
            for i in 1..10 {
                println!("hi number {i} from the first task!");
                trpl::sleep(Duration::from_millis(500)).await;
            }
        });

        for i in 1..5 {
            println!("hi number {i} from the second task!");
            trpl::sleep(Duration::from_millis(500)).await;
        }
    });
}
```

[Listagem 17-6](#listagem-17-6): Criando uma nova tarefa para imprimir uma coisa enquanto a tarefa principal imprime outra

A partir daqui, cada exemplo inclui `trpl::block_on` em `main` como acima; muitas vezes omitiremos esse trecho, como fazemos com `main` em geral.

Dois loops com `trpl::sleep` de meio segundo; um dentro de `trpl::spawn_task`, outro no `for` de nível superior, com `await` após cada `sleep`.

O comportamento é parecido com threads — inclusive a ordem das mensagens pode variar:

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Esta versão para quando o `for` do bloco async principal termina, porque a tarefa de `spawn_task` é encerrada quando `main` acaba. Para ir até o fim da tarefa, use um join handle e `await`, como na Listagem 17-7.

**Arquivo: src/main.rs**

```rust
        let handle = trpl::spawn_task(async {
            for i in 1..10 {
                println!("hi number {i} from the first task!");
                trpl::sleep(Duration::from_millis(500)).await;
            }
        });

        for i in 1..5 {
            println!("hi number {i} from the second task!");
            trpl::sleep(Duration::from_millis(500)).await;
        }

        handle.await.unwrap();
```

[Listagem 17-7](#listagem-17-7): Usando `await` com join handle para executar uma tarefa até o fim

Agora os dois loops terminam:

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Até aqui, async e threads parecem equivalentes, com `await` no lugar de `join` e `await` nos `sleep`.

A diferença maior é que não precisamos de outra thread do SO. Nem precisamos de `spawn_task`: blocos `async` viram futures anônimas; podemos colocar cada loop em um bloco `async` e usar `trpl::join` (Listagem 17-8).

Em Aguardando todas as threads terminarem, usamos `join` em `JoinHandle` de `thread::spawn`. `trpl::join` é análogo, mas para futures: recebe duas futures e produz uma nova cuja saída é uma tupla com as saídas de ambas quando _as duas_ completam.

**Arquivo: src/main.rs**

```rust
        let fut1 = async {
            for i in 1..10 {
                println!("hi number {i} from the first task!");
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let fut2 = async {
            for i in 1..5 {
                println!("hi number {i} from the second task!");
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        trpl::join(fut1, fut2).await;
```

[Listagem 17-8](#listagem-17-8): Usando `trpl::join` para aguardar duas futures anônimas

Aguardamos o resultado de `trpl::join`, não cada future em sequência.

A ordem fica sempre a mesma — bem diferente de threads e de `spawn_task` na Listagem 17-7:

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

`trpl::join` é _justo_ (_fair_): verifica cada future com a mesma frequência, alternando, sem deixar uma correr à frente se a outra estiver pronta. Com threads, o SO decide qual thread rodar e por quanto tempo. Com async Rust, o runtime decide qual tarefa verificar. (Na prática, runtimes podem usar threads por baixo; garantir justiça pode exigir mais trabalho — mas é possível!) Runtimes não precisam garantir justiça; muitas vezes oferecem APIs para escolher.

Experimente: remover o bloco `async` de um ou ambos os loops; aguardar cada bloco logo após defini-lo; envolver só o primeiro loop em `async` e aguardar após o segundo. Tente prever a saída antes de executar!

## Enviando dados entre duas tarefas com passagem de mensagens

Compartilhar dados entre futures também usa passagem de mensagens, com tipos e funções async. Tomamos um caminho um pouco diferente de Transferir dados entre threads com passagem de mensagens para destacar diferenças entre threads e futures. Na Listagem 17-9, começamos com um único bloco `async` — sem tarefa separada.

**Arquivo: src/main.rs**

```rust
        let (tx, mut rx) = trpl::channel();

        let val = String::from("hi");
        tx.send(val).unwrap();

        let received = rx.recv().await.unwrap();
        println!("received '{received}'");
```

[Listagem 17-9](#listagem-17-9): Criando um channel async e atribuindo as metades a `tx` e `rx`

Usamos `trpl::channel`, versão async do channel vários-produtores / um-consumidor do Capítulo 16. A API async difere pouco: o receptor `rx` é mutável, e `recv` produz uma future a ser aguardada. Podemos enviar sem thread ou tarefa separada — basta `await` em `rx.recv`.

`Receiver::recv` síncrono em `std::sync::mpsc` bloqueia até haver mensagem. `trpl::Receiver::recv` não bloqueia: devolve o controle ao runtime até chegar mensagem ou fechar o envio. Não aguardamos `send` porque o channel é ilimitado.

> **Nota:** Todo este código async roda num bloco dentro de `trpl::block_on`, então pode evitar bloqueio _dentro_ dele. O código _fora_ bloqueia até `block_on` retornar — é o propósito de `block_on`: escolher onde bloquear num conjunto de código async e onde passar entre sync e async.

A mensagem chega na hora. Mas, embora usemos future, ainda não há concorrência: tudo na listagem é sequencial.

Na Listagem 17-10 enviamos várias mensagens com `sleep` entre elas.

**Arquivo: src/main.rs**

```rust
        let (tx, mut rx) = trpl::channel();

        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("future"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            trpl::sleep(Duration::from_millis(500)).await;
        }

        while let Some(value) = rx.recv().await {
            println!("received '{value}'");
        }
```

[Listagem 17-10](#listagem-17-10): Enviando e recebendo várias mensagens no channel async com `await` entre cada envio

Rust ainda não tem `for` sobre séries produzidas de forma assíncrona; usamos o loop `while let` (versão em loop de `if let` do Capítulo 6 em Fluxo de controle conciso com `if let` e `let...else`). O loop continua enquanto o padrão casar com o valor.

`rx.recv` produz uma future; ao aguardar, o runtime pausa até `Some(message)` ou `None` quando o channel fecha.

Com `while let`, se `rx.recv().await` for `Some(message)`, usamos a mensagem no corpo; se for `None`, o loop termina. Cada iteração volta ao await.

As mensagens chegam todas de uma vez, ~2 segundos após o início, não a cada meio segundo. O programa também não termina — use <kbd>Ctrl</kbd>+<kbd>C</kbd>.

### Código dentro de um bloco async executa de forma linear

Dentro de um bloco async, a ordem dos `await` no código é a ordem de execução. Na Listagem 17-10 há um só bloco: tudo linear, sem concorrência. Todos os `send` e `sleep` acontecem antes do `while let` processar qualquer `recv`.

Para atraso entre mensagens, separamos envio e recepção em blocos `async` distintos e usamos `trpl::join` (Listagem 17-11). Aguardamos o resultado de `join`, não cada future em sequência.

**Arquivo: src/main.rs**

```rust
        let (tx, mut rx) = trpl::channel();

        let tx_fut = async {
            let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("future"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async {
            while let Some(value) = rx.recv().await {
                println!("received '{value}'");
            }
        };

        trpl::join(tx_fut, rx_fut).await;
```

[Listagem 17-11](#listagem-17-11): Separando `send` e `recv` em blocos `async` e aguardando com `join`

Agora as mensagens saem a cada 500 ms.

### Movendo ownership para um bloco async

O programa ainda não encerra por causa do `while let` e `trpl::join`:

- `join` só completa quando _ambas_ as futures terminam.
- `tx_fut` termina após o último `sleep` após enviar.
- `rx_fut` só termina quando o `while let` acaba.
- O loop só acaba quando `recv` retorna `None`.
- `None` só quando o channel fecha.
- Fecha com `rx.close` ou quando `tx` é descartado.
- `tx` só é descartado quando o bloco externo de `block_on` termina — mas esse bloco espera `join` completar.

O bloco de envio só _empresta_ `tx`. Com `async move` (Capturando referências ou movendo ownership e Usando closures `move` com threads), movemos `tx` para o bloco e ele é descartado ao fim (Listagem 17-12).

**Arquivo: src/main.rs**

```rust
        let (tx, mut rx) = trpl::channel();

        let tx_fut = async move {
            let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("future"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async {
            while let Some(value) = rx.recv().await {
                println!("received '{value}'");
            }
        };

        trpl::join(tx_fut, rx_fut).await;
```

[Listagem 17-12](#listagem-17-12): Revisão da Listagem 17-11 que encerra corretamente ao terminar

Esta versão encerra após a última mensagem.

### Juntando várias futures com a macro `join!`

O channel async também é multi-produtor: `clone` em `tx` e vários blocos `async move` (Listagem 17-13).

**Arquivo: src/main.rs**

```rust
        let (tx, mut rx) = trpl::channel();

        let tx1 = tx.clone();
        let tx1_fut = async move {
            let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("future"),
            ];

            for val in vals {
                tx1.send(val).unwrap();
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async {
            while let Some(value) = rx.recv().await {
                println!("received '{value}'");
            }
        };

        let tx_fut = async move {
            let vals = vec![
                String::from("more"),
                String::from("messages"),
                String::from("for"),
                String::from("you"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                trpl::sleep(Duration::from_millis(1500)).await;
            }
        };

        trpl::join!(tx1_fut, tx_fut, rx_fut);
```

[Listagem 17-13](#listagem-17-13): Vários produtores com blocos async

O que importa é a ordem em que as futures são aguardadas, não em que ordem os blocos foram escritos. Ambos os blocos de envio precisam ser `async move`. Usamos `trpl::join!` em vez de `join` para número arbitrário de futures conhecido em tempo de compilação.

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Vimos passagem de mensagens entre futures, execução linear dentro de um bloco, `move` em blocos async e juntar várias futures. Em seguida, quando e como avisar o runtime para trocar de tarefa.
