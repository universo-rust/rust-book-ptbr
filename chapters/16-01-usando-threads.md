---
title: "Usando threads para executar código simultaneamente"
chapter_code: 16-01
slug: usando-threads-para-executar-codigo-simultaneamente
---

# Usando Threads para Executar Código Simultaneamente

Na maioria dos sistemas operacionais atuais, o código de um programa executado é executado em um _processo_, e o sistema operacional gerencia vários processos de uma vez. Dentro de um programa, você também pode ter partes independentes que executam simultaneamente. Os recursos que executam essas partes independentes são chamados de _threads_. Por exemplo, um servidor web pode ter várias threads para poder responder a mais de uma requisição ao mesmo tempo.

Dividir a computação do seu programa em várias threads para executar várias tarefas ao mesmo tempo pode melhorar o desempenho, mas também adiciona complexidade. Como threads podem executar simultaneamente, não há garantia inerente sobre a ordem em que partes do seu código em threads diferentes executarão. Isso pode levar a problemas, como:

- Condições de corrida (_race conditions_), em que threads acessam dados ou recursos em ordem inconsistente
- Deadlocks, em que duas threads esperam uma pela outra, impedindo ambas de continuar
- Bugs que só acontecem em certas situações e são difíceis de reproduzir e corrigir de forma confiável

O Rust tenta mitigar os efeitos negativos do uso de threads, mas programar em um contexto multithread ainda exige pensamento cuidadoso e uma estrutura de código diferente da de programas executando em uma única thread.

Linguagens de programação implementam threads de algumas formas diferentes, e muitos sistemas operacionais fornecem uma API que a linguagem pode chamar para criar novas threads. A biblioteca padrão do Rust usa um modelo de implementação de threads _1:1_, pelo qual um programa usa uma thread do sistema operacional por uma thread da linguagem. Há crates que implementam outros modelos de threading que fazem trocas diferentes em relação ao modelo 1:1. (O sistema async do Rust, que veremos no próximo capítulo, também fornece outra abordagem à concorrência.)

## Criando uma Nova Thread com `spawn`

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos a ela uma closure (falamos sobre closures no Capítulo 13) contendo o código que queremos executar na nova thread. O exemplo na Listagem 16-1 imprime algum texto da thread principal e outro texto de uma nova thread.

**Arquivo: src/main.rs**

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }
}
```

[Listagem 16-1](#listagem-16-1): Criando uma nova thread para imprimir uma coisa enquanto a thread principal imprime outra

Note que, quando a thread principal de um programa Rust termina, todas as threads criadas são encerradas, tenham ou não terminado de executar. A saída deste programa pode ser um pouco diferente a cada vez, mas se parecerá com o seguinte:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

As chamadas a `thread::sleep` forçam uma thread a parar sua execução por um curto período, permitindo que outra thread execute. As threads provavelmente se alternarão, mas isso não é garantido: depende de como seu sistema operacional agenda as threads. Nesta execução, a thread principal imprimiu primeiro, embora a instrução de impressão da thread criada apareça primeiro no código. E embora tenhamos dito à thread criada para imprimir até `i` ser `9`, ela só chegou a `5` antes da thread principal encerrar.

Se você executar este código e só vir saída da thread principal, ou não vir sobreposição, tente aumentar os números nos intervalos para criar mais oportunidades para o sistema operacional alternar entre as threads.

## Esperando Todas as Threads Terminarem

O código na Listagem 16-1 não só para a thread criada prematuramente na maioria das vezes porque a thread principal termina, mas como não há garantia sobre a ordem em que as threads executam, também não podemos garantir que a thread criada chegará a executar!

Podemos corrigir o problema da thread criada não executar ou terminar prematuramente salvando o valor de retorno de `thread::spawn` em uma variável. O tipo de retorno de `thread::spawn` é `JoinHandle<T>`. Um `JoinHandle<T>` é um valor owned que, quando chamamos o método `join` nele, esperará sua thread terminar. A Listagem 16-2 mostra como usar o `JoinHandle<T>` da thread que criamos na Listagem 16-1 e como chamar `join` para garantir que a thread criada termine antes de `main` sair.

**Arquivo: src/main.rs**

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

[Listagem 16-2](#listagem-16-2): Salvando um `JoinHandle<T>` de `thread::spawn` para garantir que a thread execute até o fim

Chamar `join` no handle bloqueia a thread que está executando atualmente até a thread representada pelo handle terminar. _Bloquear_ uma thread significa que a thread é impedida de realizar trabalho ou sair. Como colocamos a chamada a `join` depois do loop `for` da thread principal, executar a Listagem 16-2 deve produzir saída semelhante a esta:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

As duas threads continuam se alternando, mas a thread principal espera por causa da chamada a `handle.join()` e não termina até a thread criada terminar.

Mas vejamos o que acontece quando, em vez disso, movemos `handle.join()` antes do loop `for` em `main`, assim:

**Arquivo: src/main.rs**

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }
}
```

A thread principal esperará a thread criada terminar e então executará seu loop `for`, então a saída não será mais intercalada, como mostrado aqui:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Detalhes pequenos, como onde `join` é chamado, podem afetar se suas threads executam ao mesmo tempo ou não.

## Usando Closures `move` com Threads

Costumamos usar a palavra-chave `move` com closures passadas a `thread::spawn` porque a closure então tomará ownership dos valores que usa do ambiente, transferindo assim a ownership desses valores de uma thread para outra. Em Capturando referências ou movendo ownership no Capítulo 13, discutimos `move` no contexto de closures. Agora nos concentraremos mais na interação entre `move` e `thread::spawn`.

Observe na Listagem 16-1 que a closure que passamos a `thread::spawn` não recebe argumentos: não estamos usando dados da thread principal no código da thread criada. Para usar dados da thread principal na thread criada, a closure da thread criada deve capturar os valores de que precisa. A Listagem 16-3 mostra uma tentativa de criar um vetor na thread principal e usá-lo na thread criada. No entanto, isso ainda não funcionará, como você verá em um momento.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {v:?}");
    });

    handle.join().unwrap();
}
```

[Listagem 16-3](#listagem-16-3): Tentativa de usar um vetor criado pela thread principal em outra thread

A closure usa `v`, então capturará `v` e o tornará parte do ambiente da closure. Como `thread::spawn` executa esta closure em uma nova thread, deveríamos poder acessar `v` dentro dessa nova thread. Mas quando compilamos este exemplo, obtemos o seguinte erro:

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0373]: closure may outlive the current function, but it borrows `v`, which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {v:?}");
  |                                     - `v` is borrowed here
  |
note: function requires argument type to outlive `'static`
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here's a vector: {v:?}");
8 | |     });
  | |______^
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try `rustc --explain E0373`.
error: could not compile `threads` (bin "threads") due to 1 previous error
```

O Rust _infere_ como capturar `v`, e como `println!` só precisa de uma referência a `v`, a closure tenta emprestar `v`. No entanto, há um problema: o Rust não pode dizer por quanto tempo a thread criada executará, então não sabe se a referência a `v` será sempre válida.

A Listagem 16-4 fornece um cenário mais provável de ter uma referência a `v` que não será válida.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {v:?}");
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

[Listagem 16-4](#listagem-16-4): Uma thread com uma closure que tenta capturar uma referência a `v` de uma thread principal que descarta `v`

Se o Rust nos permitisse executar este código, há a possibilidade de que a thread criada fosse imediatamente colocada em segundo plano sem executar. A thread criada tem uma referência a `v` dentro, mas a thread principal descarta `v` imediatamente, usando a função `drop` que discutimos no Capítulo 15. Depois, quando a thread criada começa a executar, `v` não é mais válido, então uma referência a ele também é inválida. Que pena!

Para corrigir o erro do compilador na Listagem 16-3, podemos usar o conselho da mensagem de erro:

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a tomar ownership dos valores que usa em vez de permitir que o Rust infira que deve emprestá-los. A modificação da Listagem 16-3 mostrada na Listagem 16-5 compilará e executará como pretendemos.

**Arquivo: src/main.rs**

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {v:?}");
    });

    handle.join().unwrap();
}
```

[Listagem 16-5](#listagem-16-5): Usando a palavra-chave `move` para forçar uma closure a tomar ownership dos valores que usa

Poderíamos ser tentados a tentar a mesma coisa para corrigir o código na Listagem 16-4, onde a thread principal chamou `drop`, usando uma closure `move`. No entanto, esta correção não funcionará porque o que a Listagem 16-4 tenta fazer é proibido por uma razão diferente. Se adicionássemos `move` à closure, moveríamos `v` para o ambiente da closure, e não poderíamos mais chamar `drop` nele na thread principal. Obteríamos este erro do compilador em vez disso:

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
 4 |     let v = vec![1, 2, 3];
   |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
 5 |
 6 |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
 7 |         println!("Here's a vector: {v:?}");
   |                                     - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move
   |
help: consider cloning the value before moving it into the closure
   |
 6 ~     let value = v.clone();
 7 ~     let handle = thread::spawn(move || {
 8 ~         println!("Here's a vector: {value:?}");
   |

For more information about this error, try `rustc --explain E0382`.
error: could not compile `threads` (bin "threads") due to 1 previous error
```

As regras de ownership do Rust nos salvaram novamente! Obtivemos um erro do código na Listagem 16-3 porque o Rust estava sendo conservador e apenas emprestando `v` para a thread, o que significava que a thread principal poderia teoricamente invalidar a referência da thread criada. Ao dizer ao Rust para mover a ownership de `v` para a thread criada, garantimos ao Rust que a thread principal não usará mais `v`. Se mudássemos a Listagem 16-4 da mesma forma, estaríamos violando as regras de ownership ao tentar usar `v` na thread principal. A palavra-chave `move` substitui o padrão conservador de empréstimo do Rust; ela não nos deixa violar as regras de ownership.

Agora que cobrimos o que são threads e os métodos fornecidos pela API de threads, vamos olhar algumas situações em que podemos usar threads.
