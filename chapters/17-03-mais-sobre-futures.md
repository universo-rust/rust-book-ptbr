---
title: "Mais sobre futures"
chapter_code: 17-03
slug: mais-sobre-futures
---

# Entregando Controle ao Runtime

Lembre-se de Nosso primeiro programa async: em cada ponto de await, o Rust dá ao runtime chance de pausar a tarefa e mudar para outra se a future aguardada não estiver pronta. O inverso também vale: Rust _só_ pausa blocos async e devolve controle ao runtime em pontos de await. Tudo entre awaits é síncrono.

Isso significa que muito trabalho num bloco async sem await bloqueia outras futures — _starvation_. Às vezes não importa; em setup caro ou tarefas indefinidas, pense em quando devolver controle ao runtime.

Simulemos operação longa com a função `slow` (Listagem 17-14).

**Arquivo: src/main.rs**

```rust
fn slow(name: &str, ms: u64) {
    thread::sleep(Duration::from_millis(ms));
    println!("'{name}' ran for {ms}ms");
}
```

<a id="listagem-17-14"></a>

[Listagem 17-14](#listagem-17-14): Usando `thread::sleep` para simular operações lentas

Usamos `std::thread::sleep` em vez de `trpl::sleep` para bloquear a thread atual — útil para representar trabalho longo e bloqueante.

Na Listagem 17-15, `slow` em um par de futures.

**Arquivo: src/main.rs**

```rust
        let a = async {
            println!("'a' started.");
            slow("a", 30);
            slow("a", 10);
            slow("a", 20);
            trpl::sleep(Duration::from_millis(50)).await;
            println!("'a' finished.");
        };

        let b = async {
            println!("'b' started.");
            slow("b", 75);
            slow("b", 10);
            slow("b", 15);
            slow("b", 350);
            trpl::sleep(Duration::from_millis(50)).await;
            println!("'b' finished.");
        };

        trpl::select(a, b).await;
```

<a id="listagem-17-15"></a>

[Listagem 17-15](#listagem-17-15): Chamando `slow` para simular operações lentas

Cada future só devolve controle após um monte de `slow`. Saída típica:

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Como na Listagem 17-5 com `select` em URLs, `select` termina quando `a` acaba — sem intercalar `slow` entre as duas futures. `a` faz todo o trabalho até o `sleep`, depois `b`, depois `a` termina. Para progresso alternado, precisamos de await entre as tarefas lentas.

Na Listagem 17-16, `trpl::sleep` entre cada `slow`.

**Arquivo: src/main.rs**

```rust
        let one_ms = Duration::from_millis(1);

        let a = async {
            println!("'a' started.");
            slow("a", 30);
            trpl::sleep(one_ms).await;
            slow("a", 10);
            trpl::sleep(one_ms).await;
            slow("a", 20);
            trpl::sleep(one_ms).await;
            println!("'a' finished.");
        };

        let b = async {
            println!("'b' started.");
            slow("b", 75);
            trpl::sleep(one_ms).await;
            slow("b", 10);
            trpl::sleep(one_ms).await;
            slow("b", 15);
            trpl::sleep(one_ms).await;
            slow("b", 350);
            trpl::sleep(one_ms).await;
            println!("'b' finished.");
        };

        trpl::select(a, b).await;
```

<a id="listagem-17-16"></a>

[Listagem 17-16](#listagem-17-16): Usando `trpl::sleep` para alternar progresso

Agora o trabalho se intercala:

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` ainda roda um pouco antes de ceder, porque chama `slow` antes do primeiro `sleep`; depois as futures alternam a cada await.

Não queremos _dormir_ de verdade — queremos progredir o mais rápido possível e só devolver controle. Use `trpl::yield_now` (Listagem 17-17).

**Arquivo: src/main.rs**

```rust
        let a = async {
            println!("'a' started.");
            slow("a", 30);
            trpl::yield_now().await;
            slow("a", 10);
            trpl::yield_now().await;
            slow("a", 20);
            trpl::yield_now().await;
            println!("'a' finished.");
        };

        let b = async {
            println!("'b' started.");
            slow("b", 75);
            trpl::yield_now().await;
            slow("b", 10);
            trpl::yield_now().await;
            slow("b", 15);
            trpl::yield_now().await;
            slow("b", 350);
            trpl::yield_now().await;
            println!("'b' finished.");
        };

        trpl::select(a, b).await;
```

<a id="listagem-17-17"></a>

[Listagem 17-17](#listagem-17-17): Usando `yield_now` para alternar progresso

Fica mais claro e pode ser bem mais rápido que `sleep`: timers costumam ter granularidade mínima (ex.: 1 ms mesmo com `Duration` de 1 ns).

Async pode ajudar até em tarefas limitadas por CPU, dependendo do resto do programa: estrutura relações entre partes do programa (com custo da máquina de estados async). É _multitarefa cooperativa_: cada future decide quando ceder via await e deve evitar bloquear demais. Em alguns SOs embarcados em Rust, é o _único_ tipo de multitarefa!

Na prática você não alterna chamada e `yield` em cada linha. Yield é barato, mas não grátis; quebrar tarefa CPU-bound pode piorar desempenho — meça. O importante é: trabalho que você esperava concorrente mas roda em série pode precisar de mais pontos de await.

## Construindo nossas próprias abstrações async

Podemos compor futures em novos padrões — por exemplo, `timeout` com blocos que já temos (Listagem 17-18).

**Arquivo: src/main.rs (Este código não compila!)**

```rust
        let slow = async {
            trpl::sleep(Duration::from_secs(5)).await;
            "Finally finished"
        };

        match timeout(slow, Duration::from_secs(2)).await {
            Ok(message) => println!("Succeeded with '{message}'"),
            Err(duration) => {
                println!("Failed after {} seconds", duration.as_secs())
            }
        }
```

<a id="listagem-17-18"></a>

[Listagem 17-18](#listagem-17-18): Usando o `timeout` imaginado com limite de tempo

API de `timeout`:

- Função async (para aguardá-la).
- Primeiro parâmetro: future genérica.
- Segundo: tempo máximo (`Duration` para passar a `trpl::sleep`).
- Retorno `Result`: `Ok` com o valor da future, ou `Err` com a duração se o tempo esgotar.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
async fn timeout<F: Future>(
    future_to_try: F,
    max_time: Duration,
) -> Result<F::Output, Duration> {
    // Aqui vai nossa implementação!
}
```

<a id="listagem-17-19"></a>

[Listagem 17-19](#listagem-17-19): Assinatura de `timeout`

Corremos a future passada contra um timer com `trpl::sleep` e `trpl::select` (Listagem 17-20).

**Arquivo: src/main.rs**

```rust
use trpl::Either;

async fn timeout<F: Future>(
    future_to_try: F,
    max_time: Duration,
) -> Result<F::Output, Duration> {
    match trpl::select(future_to_try, trpl::sleep(max_time)).await {
        Either::Left(output) => Ok(output),
        Either::Right(_) => Err(max_time),
    }
}
```

<a id="listagem-17-20"></a>

[Listagem 17-20](#listagem-17-20): Definindo `timeout` com `select` e `sleep`

`trpl::select` não é justo: sempre faz poll na ordem dos argumentos. Passamos `future_to_try` primeiro para ela ter chance mesmo com `max_time` curto. `Left(output)` → `Ok(output)`; timer primeiro → `Err(max_time)`.

Executando:

```text
Failed after 2 seconds
```

Futures compostas permitem ferramentas poderosas (timeout + retry + rede, como na Listagem 17-5). Na prática você usa `async`/`await` e, em segundo plano, `select` e `join!` para orquestrar as futures de mais alto nível.

Vimos várias formas de várias futures ao mesmo tempo. A seguir, séries ao longo do tempo com _streams_.
