---
title: "Mais sobre futures"
chapter_code: 17-03
slug: mais-sobre-futures
---

# Entregando controle ao runtime

Lembre-se da seção ["Nosso primeiro programa async"](/livro/cap17-01-futures-e-a-sintaxe-async#nosso-primeiro-programa-async): em cada ponto de `await`, o Rust dá ao runtime a chance de pausar a tarefa atual e trocar para outra, caso a future que está sendo aguardada ainda não esteja pronta. O inverso também é verdadeiro: o Rust _só_ pausa blocos async e devolve o controle ao runtime em pontos de `await`. Tudo que está entre dois pontos de `await` é síncrono.

Isso significa que, se você fizer muito trabalho dentro de um bloco async sem nenhum ponto de `await`, essa future impedirá outras futures de avançarem. Às vezes você verá isso descrito como uma future deixando outras futures sem recursos, ou causando _starvation_. Em alguns casos, isso não é um grande problema. Mas, se você estiver fazendo uma inicialização cara, um trabalho demorado ou uma tarefa que continua rodando indefinidamente, precisará pensar em quando e onde devolver o controle ao runtime.

Vamos simular uma operação demorada para enxergar o problema de _starvation_ e depois ver como resolvê-lo. A Listagem 17-14 apresenta uma função `slow`.

**Arquivo: src/main.rs**

```rust
fn slow(name: &str, ms: u64) {
    thread::sleep(Duration::from_millis(ms));
    println!("'{name}' ran for {ms}ms");
}
```

<a id="listagem-17-14"></a>

[Listagem 17-14](#listagem-17-14): Usando `thread::sleep` para simular operações lentas

Esse código usa `std::thread::sleep` em vez de `trpl::sleep`, de modo que chamar `slow` bloqueia a thread atual por alguns milissegundos. Podemos usar `slow` como substituto para operações reais que sejam demoradas e bloqueantes.

Na Listagem 17-15, usamos `slow` para imitar esse tipo de trabalho limitado por CPU em um par de futures.

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

[Listagem 17-15](#listagem-17-15): Chamando a função `slow` para simular operações lentas

Cada future só devolve o controle ao runtime _depois_ de executar várias operações lentas. Ao rodar esse código, você verá uma saída como esta:

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

Assim como na Listagem 17-5, em que usamos `trpl::select` para fazer duas futures competirem enquanto buscavam duas URLs, `select` ainda termina assim que `a` termina. Mas não há intercalação entre as chamadas a `slow` nas duas futures. A future `a` faz todo o seu trabalho até chegar ao `await` em `trpl::sleep`; depois, a future `b` faz todo o seu trabalho até chegar ao próprio `await` em `trpl::sleep`; por fim, a future `a` completa. Para permitir que as duas futures avancem entre suas tarefas lentas, precisamos de pontos de `await` nos quais possamos devolver o controle ao runtime. Em outras palavras, precisamos de algo que possamos aguardar!

Já conseguimos ver esse tipo de passagem de controle na Listagem 17-15: se removêssemos o `trpl::sleep` no fim da future `a`, ela completaria sem que a future `b` rodasse _sequer uma vez_. Vamos começar usando `trpl::sleep` para permitir que as operações alternem seu progresso, como mostra a Listagem 17-16.

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

[Listagem 17-16](#listagem-17-16): Usando `trpl::sleep` para permitir que as operações alternem progresso

Agora o trabalho das duas futures fica intercalado:

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

A future `a` ainda roda por um tempo antes de entregar o controle a `b`, porque ela chama `slow` antes de chamar `trpl::sleep` pela primeira vez. Depois disso, as futures alternam sempre que uma delas chega a um ponto de `await`. Neste exemplo, fizemos isso depois de cada chamada a `slow`, mas poderíamos dividir o trabalho da forma que fizesse mais sentido para o nosso programa.

Na verdade, porém, não queremos usar `sleep` aqui: queremos avançar o mais rápido possível. Só precisamos devolver o controle ao runtime. Podemos fazer isso diretamente com a função `trpl::yield_now`. Na Listagem 17-17, substituímos todas as chamadas a `trpl::sleep` por `trpl::yield_now`.

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

[Listagem 17-17](#listagem-17-17): Usando `yield_now` para permitir que as operações alternem progresso

Esse código deixa a intenção mais clara e pode ser bem mais rápido que usar `sleep`, porque temporizadores como o usado por `sleep` normalmente têm um limite de granularidade. A versão de `sleep` que estamos usando, por exemplo, sempre espera pelo menos um milissegundo, mesmo que passemos uma `Duration` de um nanossegundo. E computadores modernos são _rápidos_: eles conseguem fazer muita coisa em um milissegundo!

Isso significa que async pode ser útil até em tarefas limitadas por CPU, dependendo do que mais o programa está fazendo, porque oferece uma ferramenta prática para estruturar as relações entre partes diferentes do programa. O custo é o overhead da máquina de estados async. Essa é uma forma de _multitarefa cooperativa_: cada future tem o poder de decidir quando entrega o controle por meio de pontos de `await`. Por isso, cada future também tem a responsabilidade de não bloquear por tempo demais. Em alguns sistemas operacionais embarcados escritos em Rust, esse é o _único_ tipo de multitarefa disponível!

Em código real, é claro, você normalmente não alternará chamadas de função com pontos de `await` em cada linha. Entregar o controle desse jeito é relativamente barato, mas não é gratuito. Em muitos casos, tentar quebrar uma tarefa limitada por CPU pode torná-la bem mais lenta; às vezes, para o desempenho _geral_, é melhor deixar uma operação bloquear por pouco tempo. Meça sempre para descobrir onde estão os gargalos reais do seu código. Ainda assim, vale guardar essa dinâmica: se muito trabalho que você esperava ver concorrente está acontecendo em série, talvez faltem pontos de `await`.

## Construindo nossas próprias abstrações async

Também podemos compor futures para criar novos padrões. Por exemplo, podemos construir uma função `timeout` usando os blocos async que já conhecemos. Quando terminarmos, o resultado será mais um bloco de construção que poderemos usar para criar outras abstrações async.

A Listagem 17-18 mostra como esperamos que esse `timeout` funcione com uma future lenta.

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

[Listagem 17-18](#listagem-17-18): Usando o `timeout` imaginado para executar uma operação lenta com limite de tempo

Vamos implementar isso! Para começar, pensemos na API de `timeout`:

- Ela precisa ser uma função async, para que possamos aguardá-la.
- O primeiro parâmetro deve ser a future que queremos executar. Podemos torná-la genérica para aceitar qualquer future.
- O segundo parâmetro será o tempo máximo de espera. Usar uma `Duration` facilita passá-la para `trpl::sleep`.
- Ela deve retornar um `Result`. Se a future terminar com sucesso, o `Result` será `Ok` com o valor produzido pela future. Se o tempo esgotar primeiro, o `Result` será `Err` com a duração usada como limite.

A Listagem 17-19 mostra essa declaração.

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

[Listagem 17-19](#listagem-17-19): Definindo a assinatura de `timeout`

Isso satisfaz nossos objetivos de tipos. Agora precisamos pensar no _comportamento_: queremos fazer a future recebida competir contra a duração máxima. Podemos usar `trpl::sleep` para criar uma future temporizadora a partir da duração e `trpl::select` para executar esse temporizador junto com a future passada por quem chamou a função.

Na Listagem 17-20, implementamos `timeout` fazendo `match` no resultado de aguardar `trpl::select`.

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

A implementação de `trpl::select` não é justa: ela sempre faz _poll_ dos argumentos na ordem em que eles são passados. Outras implementações de `select` podem escolher aleatoriamente qual argumento verificar primeiro. Por isso, passamos `future_to_try` primeiro, para que ela tenha chance de completar mesmo quando `max_time` for uma duração muito curta. Se `future_to_try` terminar primeiro, `select` retornará `Left` com a saída dessa future. Se o temporizador terminar primeiro, `select` retornará `Right` com a saída do temporizador, que é `()`.

Se `future_to_try` tiver sucesso e recebermos `Left(output)`, retornamos `Ok(output)`. Se o temporizador de `sleep` expirar primeiro e recebermos `Right(())`, ignoramos o `()` com `_` e retornamos `Err(max_time)`.

Com isso, temos um `timeout` funcional construído a partir de dois outros ajudantes async. Ao executar o código, ele imprimirá o caso de falha depois do limite de tempo:

```text
Failed after 2 seconds
```

Como futures podem ser compostas com outras futures, você consegue construir ferramentas bem poderosas a partir de blocos async menores. Por exemplo, pode usar essa mesma abordagem para combinar timeouts com novas tentativas e, por sua vez, usar isso com operações como chamadas de rede, como as da Listagem 17-5.

Na prática, você normalmente trabalhará diretamente com `async` e `await` e, em segundo plano, com funções como `select` e macros como `join!` para controlar como as futures mais externas são executadas.

Agora vimos várias formas de trabalhar com múltiplas futures ao mesmo tempo. A seguir, veremos como trabalhar com múltiplas futures em uma sequência ao longo do tempo usando _streams_.
