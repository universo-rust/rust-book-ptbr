---
title: "Transferir dados entre threads com passagem de mensagens"
chapter_code: 16-02
slug: transferir-dados-entre-threads-com-passagem-de-mensagens
---

# Transferir dados entre threads com passagem de mensagens

Uma abordagem cada vez mais popular para garantir concorrência segura é a passagem de mensagens, em que threads ou atores se comunicam enviando mensagens umas às outras contendo dados. Aqui está a ideia em um slogan da [documentação da linguagem Go](https://golang.org/doc/effective_go.html#concurrency): “Não comunique compartilhando memória; em vez disso, compartilhe memória comunicando.”

Para realizar concorrência por envio de mensagens, a biblioteca padrão do Rust fornece uma implementação de channels. Um _channel_ é um conceito geral de programação pelo qual dados são enviados de uma thread para outra.

Você pode imaginar um channel na programação como sendo semelhante a um canal direcional de água, como um riacho ou um rio. Se você colocar algo como um pato de borracha em um rio, ele viajará rio abaixo até o fim do curso d’água.

Um channel tem duas metades: um transmissor e um receptor. A metade transmissora é o local rio acima onde você coloca o pato de borracha no rio, e a metade receptora é onde o pato de borracha termina rio abaixo. Uma parte do seu código chama métodos no transmissor com os dados que quer enviar, e outra parte verifica a extremidade receptora em busca de mensagens que chegam. Diz-se que um channel está _fechado_ se qualquer uma das metades transmissor ou receptor for descartada.

Aqui, construiremos um programa que tem uma thread para gerar valores e enviá-los por um channel, e outra thread que receberá os valores e os imprimirá. Enviaremos valores simples entre threads usando um channel para ilustrar o recurso. Depois que você estiver familiarizado com a técnica, poderá usar channels para quaisquer threads que precisem se comunicar, como um sistema de chat ou um sistema em que muitas threads executam partes de um cálculo e enviam as partes para uma thread que agrega os resultados.

Primeiro, na Listagem 16-6, criaremos um channel, mas não faremos nada com ele.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

<a id="listagem-16-6"></a>

[Listagem 16-6](#listagem-16-6): Criando um channel e atribuindo as duas metades a `tx` e `rx`

Criamos um novo channel usando a função `mpsc::channel`; `mpsc` significa _multiple producer, single consumer_ (vários produtores, um consumidor). Em resumo, a forma como a biblioteca padrão do Rust implementa channels significa que um channel pode ter várias extremidades _enviadoras_ que produzem valores, mas apenas uma extremidade _receptora_ que consome esses valores. Imagine vários riachos fluindo juntos em um grande rio: tudo enviado por qualquer um dos riachos acabará em um rio no final. Começaremos com um único produtor por agora, mas adicionaremos vários produtores quando este exemplo estiver funcionando.

A função `mpsc::channel` retorna uma tupla, cujo primeiro elemento é a extremidade enviadora — o transmissor — e o segundo elemento é a extremidade receptora — o receptor. As abreviações `tx` e `rx` são tradicionalmente usadas em muitos campos para _transmitter_ e _receiver_, respectivamente, então nomeamos nossas variáveis assim para indicar cada extremidade. Estamos usando uma instrução `let` com um pattern que desestrutura as tuplas; discutiremos o uso de patterns em instruções `let` e desestruturação no Capítulo 19. Por agora, saiba que usar uma instrução `let` desta forma é uma abordagem conveniente para extrair as partes da tupla retornada por `mpsc::channel`.

Vamos mover a extremidade transmissora para uma thread criada e fazer com que ela envie uma string para que a thread criada se comunique com a thread principal, como mostrado na Listagem 16-7. É como colocar um pato de borracha no rio rio acima ou enviar uma mensagem de chat de uma thread para outra.

**Arquivo: src/main.rs**

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

<a id="listagem-16-7"></a>

[Listagem 16-7](#listagem-16-7): Movendo `tx` para uma thread criada e enviando `"hi"`

Novamente, estamos usando `thread::spawn` para criar uma nova thread e então `move` para mover `tx` para a closure para que a thread criada possua `tx`. A thread criada precisa possuir o transmissor para poder enviar mensagens pelo channel.

O transmissor tem um método `send` que recebe o valor que queremos enviar. O método `send` retorna um tipo `Result<T, E>`, então se o receptor já foi descartado e não há para onde enviar um valor, a operação de envio retornará um erro. Neste exemplo, estamos chamando `unwrap` para entrar em pânico em caso de erro. Mas em uma aplicação real, trataríamos isso adequadamente: volte ao Capítulo 9 para revisar estratégias de tratamento de erros adequado.

Na Listagem 16-8, obteremos o valor do receptor na thread principal. É como recuperar o pato de borracha da água no final do rio ou receber uma mensagem de chat.

**Arquivo: src/main.rs**

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

<a id="listagem-16-8"></a>

[Listagem 16-8](#listagem-16-8): Recebendo o valor `"hi"` na thread principal e imprimindo-o

O receptor tem dois métodos úteis: `recv` e `try_recv`. Estamos usando `recv`, abreviação de _receive_ (receber), que bloqueará a execução da thread principal e esperará até que um valor seja enviado pelo channel. Quando um valor é enviado, `recv` o retornará em um `Result<T, E>`. Quando o transmissor fecha, `recv` retornará um erro para sinalizar que não virão mais valores.

O método `try_recv` não bloqueia, mas em vez disso retorna imediatamente um `Result<T, E>`: um valor `Ok` contendo uma mensagem se houver uma disponível e um valor `Err` se não houver mensagens desta vez. Usar `try_recv` é útil se esta thread tiver outro trabalho a fazer enquanto espera mensagens: poderíamos escrever um loop que chama `try_recv` de vez em quando, trata uma mensagem se houver uma disponível e, caso contrário, faz outro trabalho por um pouco antes de verificar novamente.

Usamos `recv` neste exemplo por simplicidade; não temos outro trabalho para a thread principal fazer além de esperar mensagens, então bloquear a thread principal é apropriado.

Quando executamos o código da Listagem 16-8, veremos o valor impresso da thread principal:

```text
Got: hi
```

Perfeito!

## Transferindo ownership através de channels

As regras de ownership desempenham um papel vital no envio de mensagens porque ajudam você a escrever código concorrente seguro. Prevenir erros em programação concorrente é a vantagem de pensar em ownership em todo o seu programa Rust. Vamos fazer um experimento para mostrar como channels e ownership trabalham juntos para prevenir problemas: tentaremos usar um valor `val` na thread criada _depois_ de tê-lo enviado pelo channel. Tente compilar o código da Listagem 16-9 para ver por que este código não é permitido.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {val}");
    });

    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

<a id="listagem-16-9"></a>

[Listagem 16-9](#listagem-16-9): Tentativa de usar `val` depois de enviá-lo pelo channel

Aqui, tentamos imprimir `val` depois de enviá-lo pelo channel via `tx.send`. Permitir isso seria uma má ideia: depois que o valor foi enviado para outra thread, essa thread poderia modificá-lo ou descartá-lo antes de tentarmos usar o valor novamente. Potencialmente, as modificações da outra thread poderiam causar erros ou resultados inesperados devido a dados inconsistentes ou inexistentes. No entanto, o Rust nos dá um erro se tentarmos compilar o código da Listagem 16-9:

```console
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: `val`
  --> src/main.rs:10:27
   |
 8 |         let val = String::from("hi");
   |             --- move occurs because `val` has type `String`, which does not implement the `Copy` trait
 9 |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {val}");
   |                           ^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `message-passing` (bin "message-passing") due to 1 previous error
```

Nosso erro de concorrência causou um erro em tempo de compilação. A função `send` toma ownership de seu parâmetro, e quando o valor é movido o receptor toma ownership dele. Isso nos impede de usar acidentalmente o valor novamente depois de enviá-lo; o sistema de ownership verifica que está tudo certo.

## Enviando vários valores

O código da Listagem 16-8 compilou e executou, mas não mostrou claramente que duas threads separadas estavam conversando pelo channel.

Na Listagem 16-10, fizemos algumas modificações que provarão que o código da Listagem 16-8 está executando concorrentemente: a thread criada agora enviará várias mensagens e pausará um segundo entre cada mensagem.

**Arquivo: src/main.rs**

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {received}");
    }
}
```

<a id="listagem-16-10"></a>

[Listagem 16-10](#listagem-16-10): Enviando várias mensagens e pausando entre cada uma

Desta vez, a thread criada tem um vetor de strings que queremos enviar para a thread principal. Iteramos sobre elas, enviando cada uma individualmente, e pausamos entre cada uma chamando a função `thread::sleep` com um valor `Duration` de um segundo.

Na thread principal, não estamos mais chamando a função `recv` explicitamente: em vez disso, tratamos `rx` como um iterator. Para cada valor recebido, imprimimos. Quando o channel é fechado, a iteração terminará.

Ao executar o código da Listagem 16-10, você deve ver a seguinte saída com uma pausa de um segundo entre cada linha:

```text
Got: hi
Got: from
Got: the
Got: thread
```

Como não temos código que pausa ou atrasa no loop `for` da thread principal, podemos dizer que a thread principal está esperando receber valores da thread criada.

## Criando vários produtores

Mencionamos anteriormente que `mpsc` era um acrônimo de _multiple producer, single consumer_. Vamos colocar `mpsc` em uso e expandir o código da Listagem 16-10 para criar várias threads que enviam valores ao mesmo receptor. Podemos fazer isso clonando o transmissor, como mostrado na Listagem 16-11.

**Arquivo: src/main.rs**

```rust
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {received}");
    }

    // --snip--
```

<a id="listagem-16-11"></a>

[Listagem 16-11](#listagem-16-11): Enviando várias mensagens de vários produtores

Desta vez, antes de criar a primeira thread criada, chamamos `clone` no transmissor. Isso nos dará um novo transmissor que podemos passar para a primeira thread criada. Passamos o transmissor original para uma segunda thread criada. Isso nos dá duas threads, cada uma enviando mensagens diferentes para o único receptor.

Quando você executar o código, sua saída deve se parecer com isto:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Você pode ver os valores em outra ordem, dependendo do seu sistema. Isso é o que torna a concorrência interessante e também difícil. Se você experimentar com `thread::sleep`, dando valores variados nas diferentes threads, cada execução será mais não determinística e criará saída diferente a cada vez.

Agora que vimos como channels funcionam, vamos olhar um método diferente de concorrência.
