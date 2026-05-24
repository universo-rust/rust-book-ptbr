---
title: "Estado compartilhado"
chapter_code: 16-03
slug: estado-compartilhado
---

# Concorrência de Estado Compartilhado

Passagem de mensagens é uma boa forma de lidar com concorrência, mas não é a única. Outro método seria várias threads acessarem os mesmos dados compartilhados. Considere novamente parte do slogan da documentação da linguagem Go: “Não comunique compartilhando memória.”

Como seria comunicar compartilhando memória? Além disso, por que entusiastas de passagem de mensagens alertam contra o uso de compartilhamento de memória?

De certa forma, channels em qualquer linguagem de programação são semelhantes a ownership única porque, depois de transferir um valor por um channel, você não deve mais usar esse valor. Concorrência de memória compartilhada é como múltipla ownership: várias threads podem acessar o mesmo local de memória ao mesmo tempo. Como você viu no Capítulo 15, onde smart pointers tornaram múltipla ownership possível, múltipla ownership pode adicionar complexidade porque esses diferentes donos precisam ser gerenciados. O sistema de tipos e as regras de ownership do Rust ajudam muito a fazer essa gestão corretamente. Como exemplo, vamos olhar mutexes, um dos primitivos de concorrência mais comuns para memória compartilhada.

## Controlando Acesso com Mutexes

_Mutex_ é abreviação de _mutual exclusion_ (exclusão mútua): um mutex permite que apenas uma thread acesse alguns dados em um dado momento. Para acessar os dados em um mutex, uma thread deve primeiro sinalizar que quer acesso pedindo para adquirir o _lock_ do mutex. O _lock_ é uma estrutura de dados que faz parte do mutex e rastreia quem tem acesso exclusivo aos dados no momento. Portanto, diz-se que o mutex _guarda_ os dados que contém por meio do sistema de locking.

Mutexes têm fama de serem difíceis de usar porque você precisa lembrar duas regras:

1. Você deve tentar adquirir o lock antes de usar os dados.
2. Quando terminar com os dados que o mutex guarda, deve liberar o lock para que outras threads possam adquiri-lo.

Como metáfora do mundo real para um mutex, imagine um painel de discussão em uma conferência com apenas um microfone. Antes de um palestrante falar, ele precisa pedir ou sinalizar que quer usar o microfone. Quando obtém o microfone, pode falar pelo tempo que quiser e então entregar o microfone ao próximo palestrante que pedir para falar. Se um palestrante esquecer de entregar o microfone quando terminar, ninguém mais consegue falar. Se a gestão do microfone compartilhado der errado, o painel não funcionará como planejado!

Gerenciar mutexes pode ser incrivelmente difícil de acertar, por isso tantas pessoas preferem channels. No entanto, graças ao sistema de tipos e às regras de ownership do Rust, você não pode errar ao fazer lock e unlock.

### A API de `Mutex<T>`

Como exemplo de como usar um mutex, comecemos usando um mutex em um contexto de thread única, como mostrado na Listagem 16-12.

**Arquivo: src/main.rs**

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {m:?}");
}
```

[Listagem 16-12](#listagem-16-12): Explorando a API de `Mutex<T>` em contexto de thread única, por simplicidade

Como muitos tipos, criamos um `Mutex<T>` usando a função associada `new`. Para acessar os dados dentro do mutex, usamos o método `lock` para adquirir o lock. Essa chamada bloqueará a thread atual para que ela não possa fazer nenhum trabalho até ser nossa vez de ter o lock.

A chamada a `lock` falharia se outra thread que mantém o lock entrasse em pânico. Nesse caso, ninguém conseguiria obter o lock, então escolhemos `unwrap` e fazemos esta thread entrar em pânico se estivermos nessa situação.

Depois de adquirir o lock, podemos tratar o valor retornado, chamado `num` neste caso, como uma referência mutável aos dados internos. O sistema de tipos garante que adquirimos um lock antes de usar o valor em `m`. O tipo de `m` é `Mutex<i32>`, não `i32`, então _devemos_ chamar `lock` para poder usar o valor `i32`. Não podemos esquecer; o sistema de tipos não nos deixa acessar o `i32` interno de outra forma.

A chamada a `lock` retorna um tipo chamado `MutexGuard`, embrulhado em um `LockResult` que tratamos com a chamada a `unwrap`. O tipo `MutexGuard` implementa `Deref` para apontar para nossos dados internos; o tipo também tem uma implementação de `Drop` que libera o lock automaticamente quando um `MutexGuard` sai de escopo, o que acontece no final do escopo interno. Como resultado, não corremos o risco de esquecer de liberar o lock e bloquear o mutex para uso por outras threads, porque a liberação do lock acontece automaticamente.

Depois de liberar o lock, podemos imprimir o valor do mutex e ver que conseguimos alterar o `i32` interno para `6`.

### Acesso compartilhado a `Mutex<T>`

Agora vamos tentar compartilhar um valor entre várias threads usando `Mutex<T>`. Criaremos 10 threads e faremos cada uma incrementar um contador em 1, para que o contador vá de 0 a 10. O exemplo na Listagem 16-13 terá um erro de compilação, e usaremos esse erro para aprender mais sobre o uso de `Mutex<T>` e como o Rust nos ajuda a usá-lo corretamente.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

[Listagem 16-13](#listagem-16-13): Dez threads, cada uma incrementando um contador guardado por um `Mutex<T>`

Criamos uma variável `counter` para guardar um `i32` dentro de um `Mutex<T>`, como fizemos na Listagem 16-12. Em seguida, criamos 10 threads iterando sobre um intervalo de números. Usamos `thread::spawn` e damos a todas as threads a mesma closure: uma que move o contador para a thread, adquire um lock no `Mutex<T>` chamando o método `lock` e então adiciona 1 ao valor no mutex. Quando uma thread termina de executar sua closure, `num` sai de escopo e libera o lock para que outra thread possa adquiri-lo.

Na thread principal, coletamos todos os join handles. Então, como fizemos na Listagem 16-2, chamamos `join` em cada handle para garantir que todas as threads terminem. Nesse ponto, a thread principal adquirirá o lock e imprimirá o resultado deste programa.

Insinuamos que este exemplo não compilaria. Agora vamos descobrir por quê!

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: borrow of moved value: `counter`
  --> src/main.rs:21:29
   |
 5 |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `std::sync::Mutex<i32>`, which does not implement the `Copy` trait
...
 8 |     for _ in 0..10 {
   |     -------------- inside of this loop
 9 |         let handle = thread::spawn(move || {
   |                                    ------- value moved into closure here, in previous iteration of loop
...
21 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value borrowed here after move
   |
help: consider moving the expression out of the loop so it is only moved once
   |
 8 ~     let mut value = counter.lock();
 9 ~     for _ in 0..10 {
10 |         let handle = thread::spawn(move || {
11 ~             let mut num = value.unwrap();
   |

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` (bin "shared-state") due to 1 previous error
```

A mensagem de erro diz que o valor `counter` foi movido na iteração anterior do loop. O Rust está nos dizendo que não podemos mover a ownership do `counter` para várias threads. Vamos corrigir o erro de compilação com o método de múltipla ownership que discutimos no Capítulo 15.

### Múltipla ownership com várias threads

No Capítulo 15, demos um valor a vários donos usando o smart pointer `Rc<T>` para criar um valor com contagem de referências. Vamos fazer o mesmo aqui e ver o que acontece. Envolveremos o `Mutex<T>` em `Rc<T>` na Listagem 16-14 e clonaremos o `Rc<T>` antes de mover a ownership para a thread.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

[Listagem 16-14](#listagem-16-14): Tentativa de usar `Rc<T>` para permitir que várias threads possuam o `Mutex<T>`

Mais uma vez, compilamos e obtemos... erros diferentes! O compilador está nos ensinando muito:

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: `Rc<std::sync::Mutex<i32>>` cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this `{closure@src/main.rs:11:36: 11:43}`
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ `Rc<std::sync::Mutex<i32>>` cannot be sent between threads safely
   |
   = help: within `{closure@src/main.rs:11:36: 11:43}`, the trait `Send` is not implemented for `Rc<std::sync::Mutex<i32>>`
note: required because it's used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in `spawn`
  --> /rustc/1159e78c4747b02ef996e55082b704c09b970588/library/std/src/thread/mod.rs:723:1

For more information about this error, try `rustc --explain E0277`.
error: could not compile `shared-state` (bin "shared-state") due to 1 previous error
```

Uau, essa mensagem de erro é bem longa! Aqui está a parte importante para focar: `` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. O compilador também nos diz o motivo: `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. Falaremos sobre `Send` na próxima seção: é uma das traits que garante que os tipos que usamos com threads são adequados para uso em situações concorrentes.

Infelizmente, `Rc<T>` não é seguro para compartilhar entre threads. Quando `Rc<T>` gerencia a contagem de referências, adiciona à contagem a cada chamada a `clone` e subtrai da contagem quando cada clone é descartado. Mas não usa primitivos de concorrência para garantir que alterações na contagem não possam ser interrompidas por outra thread. Isso poderia levar a contagens erradas — bugs sutis que por sua vez poderiam causar vazamentos de memória ou um valor ser descartado antes de terminarmos de usá-lo. O que precisamos é de um tipo exatamente como `Rc<T>`, mas que faça alterações na contagem de referências de forma thread-safe.

### Contagem de referência atômica com `Arc<T>`

Felizmente, `Arc<T>` _é_ um tipo como `Rc<T>` que é seguro para usar em situações concorrentes. O _a_ significa _atomic_ (atômico): é um tipo de contagem de referências _atomicamente_. Átomos são outro tipo de primitivo de concorrência que não cobriremos em detalhes aqui: consulte a documentação da biblioteca padrão em [`std::sync::atomic`](https://doc.rust-lang.org/std/sync/atomic/index.html) para mais detalhes. Neste ponto, você só precisa saber que átomos funcionam como tipos primitivos, mas são seguros para compartilhar entre threads.

Você pode então se perguntar por que todos os tipos primitivos não são atômicos e por que os tipos da biblioteca padrão não são implementados para usar `Arc<T>` por padrão. O motivo é que thread-safety vem com uma penalidade de desempenho que você só quer pagar quando realmente precisa. Se você está apenas realizando operações em valores dentro de uma única thread, seu código pode executar mais rápido se não tiver que impor as garantias que átomos fornecem.

Voltemos ao nosso exemplo: `Arc<T>` e `Rc<T>` têm a mesma API, então corrigimos nosso programa alterando a linha `use`, a chamada a `new` e a chamada a `clone`. O código da Listagem 16-15 finalmente compilará e executará.

**Arquivo: src/main.rs**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

[Listagem 16-15](#listagem-16-15): Usando um `Arc<T>` para envolver o `Mutex<T>` e compartilhar ownership entre várias threads

Este código imprimirá o seguinte:

```text
Result: 10
```

Conseguimos! Contamos de 0 a 10, o que pode não parecer muito impressionante, mas nos ensinou muito sobre `Mutex<T>` e thread-safety. Você também poderia usar a estrutura deste programa para fazer operações mais complexas do que apenas incrementar um contador. Usando esta estratégia, você pode dividir um cálculo em partes independentes, distribuir essas partes entre threads e então usar um `Mutex<T>` para que cada thread atualize o resultado final com sua parte.

Note que, se você está fazendo operações numéricas simples, há tipos mais simples que `Mutex<T>` fornecidos pelo módulo [`std::sync::atomic`](https://doc.rust-lang.org/std/sync/atomic/index.html) da biblioteca padrão. Esses tipos fornecem acesso seguro, concorrente e atômico a tipos primitivos. Escolhemos usar `Mutex<T>` com um tipo primitivo neste exemplo para podermos nos concentrar em como `Mutex<T>` funciona.

## Comparando `RefCell<T>`/`Rc<T>` e `Mutex<T>`/`Arc<T>`

Você pode ter notado que `counter` é imutável, mas que conseguimos obter uma referência mutável ao valor interno; isso significa que `Mutex<T>` fornece mutabilidade interior, como a família `Cell`. Da mesma forma que usamos `RefCell<T>` no Capítulo 15 para permitir mutar conteúdos dentro de um `Rc<T>`, usamos `Mutex<T>` para mutar conteúdos dentro de um `Arc<T>`.

Outro detalhe a notar é que o Rust não pode protegê-lo de todos os tipos de erros lógicos quando você usa `Mutex<T>`. Lembre-se do Capítulo 15 de que usar `Rc<T>` trazia o risco de criar ciclos de referência, em que dois valores `Rc<T>` se referem um ao outro, causando vazamentos de memória. Da mesma forma, `Mutex<T>` traz o risco de criar _deadlocks_. Estes ocorrem quando uma operação precisa fazer lock em dois recursos e duas threads adquiriram cada uma um dos locks, fazendo-as esperar uma pela outra para sempre. Se você se interessar por deadlocks, tente criar um programa Rust que tenha um deadlock; depois, pesquise estratégias de mitigação de deadlock para mutexes em qualquer linguagem e tente implementá-las em Rust. A documentação da API da biblioteca padrão para `Mutex<T>` e `MutexGuard` oferece informações úteis.

Encerraremos este capítulo falando sobre as traits `Send` e `Sync` e como podemos usá-las com tipos personalizados.
