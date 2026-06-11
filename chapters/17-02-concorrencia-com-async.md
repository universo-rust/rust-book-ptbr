---
title: "Aplicando concorrência com async"
chapter_code: 17-02
slug: aplicando-concorrencia-com-async
---

# Aplicando concorrência com async

Nesta seção, vamos aplicar `async` a alguns dos mesmos problemas de concorrência que enfrentamos com threads no Capítulo 16. Como muitas das ideias principais já apareceram lá, aqui vamos nos concentrar no que muda quando trocamos threads por futures.

Em muitos casos, as APIs para trabalhar com concorrência usando `async` lembram bastante as APIs baseadas em threads. Em outros, elas acabam sendo bem diferentes. E mesmo quando a aparência é parecida, o comportamento costuma mudar; as características de desempenho quase sempre mudam também.

## Criando uma nova tarefa com `spawn_task`

A primeira coisa que fizemos em [Criando uma nova thread com `spawn`](/livro/cap16-01-usando-threads-para-executar-codigo-simultaneamente#criando-uma-nova-thread-com-spawn) foi contar ao mesmo tempo em duas threads separadas. Vamos fazer o mesmo usando `async`. O crate `trpl` fornece uma função `spawn_task`, muito parecida com a API `thread::spawn`, e uma função `sleep`, que é uma versão assíncrona de `thread::sleep`. Juntas, elas nos permitem implementar o exemplo de contagem da Listagem 17-6.

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

<a id="listagem-17-6"></a>

[Listagem 17-6](#listagem-17-6): Criando uma nova tarefa para imprimir uma mensagem enquanto a tarefa principal imprime outra

Como ponto de partida, configuramos a função `main` com `trpl::block_on`, para que nossa função de nível mais alto possa executar código assíncrono.

> **Nota:** Daqui em diante, todos os exemplos deste capítulo usam esse mesmo envoltório com `trpl::block_on` dentro de `main`. Por isso, muitas vezes vamos omiti-lo, assim como costumamos omitir `main` em outros exemplos. Lembre-se de incluí-lo no seu código!

Dentro desse bloco, escrevemos dois loops. Cada um chama `trpl::sleep`, que espera meio segundo (500 milissegundos) antes da próxima mensagem. Um loop fica dentro de `trpl::spawn_task`; o outro fica em um `for` no nível principal do bloco. Também colocamos `await` depois de cada chamada a `sleep`.

Esse código se comporta de modo parecido com a versão baseada em threads, inclusive no fato de que as mensagens podem aparecer em uma ordem diferente no seu terminal:

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

Esta versão para assim que o loop `for` do bloco async principal termina, porque a tarefa criada por `spawn_task` é encerrada quando a função `main` termina. Se quisermos que ela rode até o final, precisamos usar um _join handle_ para esperar a primeira tarefa terminar. Com threads, usamos o método `join` para bloquear até a thread terminar. Na Listagem 17-7, podemos usar `await` para fazer a mesma coisa, porque o próprio handle da tarefa é uma future. O tipo `Output` dele é um `Result`, então também chamamos `unwrap` depois de aguardá-lo.

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

<a id="listagem-17-7"></a>

[Listagem 17-7](#listagem-17-7): Usando `await` com um _join handle_ para executar uma tarefa até o fim

Com essa versão atualizada, o programa roda até os dois loops terminarem:

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

Até aqui, parece que async e threads nos dão resultados parecidos, apenas com uma sintaxe diferente: usamos `await` no lugar de chamar `join` no handle, e também aguardamos as chamadas a `sleep`.

A diferença mais importante é que não precisamos criar outra thread do sistema operacional para fazer isso. Na verdade, neste exemplo nem precisamos criar uma tarefa separada. Como blocos `async` são compilados para futures anônimas, podemos colocar cada loop em seu próprio bloco `async` e pedir ao runtime que execute ambos até o fim usando a função `trpl::join`.

Na seção "Aguardando todas as threads terminarem", do Capítulo 16, mostramos como usar o método `join` no tipo `JoinHandle` retornado por `std::thread::spawn`. A função `trpl::join` é parecida, mas trabalha com futures. Quando passamos duas futures para ela, recebemos uma nova future cuja saída é uma tupla com a saída de cada uma das futures originais, depois que ambas terminarem. Assim, na Listagem 17-8, usamos `trpl::join` para esperar `fut1` e `fut2` terminarem. Não aguardamos `fut1` e `fut2` separadamente; aguardamos a nova future produzida por `trpl::join`. Ignoramos o resultado porque ele é apenas uma tupla com dois valores unitários.

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

<a id="listagem-17-8"></a>

[Listagem 17-8](#listagem-17-8): Usando `trpl::join` para aguardar duas futures anônimas

Ao executar esse código, vemos que as duas futures rodam até o fim:

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

Agora a ordem será exatamente a mesma toda vez, bem diferente do que vimos com threads e com `trpl::spawn_task` na Listagem 17-7. Isso acontece porque `trpl::join` é _justo_ (_fair_): ele verifica cada future com a mesma frequência, alternando entre elas, e não deixa uma disparar na frente se a outra também estiver pronta. Com threads, o sistema operacional decide qual thread executar e por quanto tempo. Com async em Rust, quem decide qual tarefa verificar é o runtime. Na prática, os detalhes podem ficar mais complexos, porque um runtime async pode usar threads do sistema operacional por baixo para gerenciar a concorrência; por isso, garantir justiça pode exigir mais trabalho do runtime, mas ainda é possível. Os runtimes não são obrigados a garantir justiça em todas as operações e muitas vezes oferecem APIs diferentes para você escolher se quer ou não esse comportamento.

Experimente algumas variações na forma de aguardar as futures e observe o que acontece:

- Remova o bloco `async` de um ou dos dois loops.
- Aguarde cada bloco `async` imediatamente depois de defini-lo.
- Coloque apenas o primeiro loop em um bloco `async` e aguarde a future resultante depois do corpo do segundo loop.

Como desafio extra, tente prever qual será a saída de cada caso antes de executar o código!

## Enviando dados entre duas tarefas com passagem de mensagens

Compartilhar dados entre futures também deve parecer familiar: vamos usar passagem de mensagens de novo, agora com versões assíncronas dos tipos e funções. Seguiremos um caminho um pouco diferente daquele da seção "Transferir dados entre threads com passagem de mensagens", no Capítulo 16, para destacar algumas diferenças importantes entre a concorrência baseada em threads e a concorrência baseada em futures. Na Listagem 17-9, começamos com um único bloco `async`, sem criar uma tarefa separada como fizemos ao criar uma thread separada.

**Arquivo: src/main.rs**

```rust
let (tx, mut rx) = trpl::channel();

let val = String::from("hi");
tx.send(val).unwrap();

let received = rx.recv().await.unwrap();
println!("received '{received}'");
```

<a id="listagem-17-9"></a>

[Listagem 17-9](#listagem-17-9): Criando um channel async e atribuindo suas duas metades a `tx` e `rx`

Aqui usamos `trpl::channel`, uma versão assíncrona da API de channel com vários produtores e um único consumidor que usamos com threads no Capítulo 16. A versão async da API muda pouco em relação à versão baseada em threads: ela usa um receptor `rx` mutável, em vez de imutável, e o método `recv` produz uma future que precisamos aguardar, em vez de produzir o valor diretamente. Agora podemos enviar mensagens do transmissor para o receptor. Repare que não precisamos criar outra thread, nem mesmo outra tarefa; basta aguardar a chamada `rx.recv`.

O método síncrono `Receiver::recv`, de `std::sync::mpsc`, bloqueia até receber uma mensagem. O método `trpl::Receiver::recv` não bloqueia, porque é assíncrono. Em vez disso, ele devolve o controle ao runtime até que uma mensagem seja recebida ou até que o lado de envio do channel seja fechado. Em contrapartida, não usamos `await` na chamada a `send`, porque ela não bloqueia. Ela não precisa bloquear porque o channel usado aqui é ilimitado.

> **Nota:** Como todo esse código async roda dentro de um bloco `async` passado para `trpl::block_on`, tudo dentro dele pode evitar bloqueios. O código fora desse bloco, porém, fica bloqueado até `block_on` retornar. Esse é justamente o papel de `trpl::block_on`: permitir que você escolha onde bloquear um conjunto de código async e, portanto, onde fazer a transição entre código síncrono e assíncrono.

Observe duas coisas nesse exemplo. Primeiro, a mensagem chega imediatamente. Segundo, embora estejamos usando uma future, ainda não há concorrência. Tudo na listagem acontece em sequência, como aconteceria se não houvesse futures envolvidas.

Vamos resolver a primeira parte enviando uma sequência de mensagens e dormindo entre elas, como mostra a Listagem 17-10.

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

<a id="listagem-17-10"></a>

[Listagem 17-10](#listagem-17-10): Enviando e recebendo várias mensagens pelo channel async, com um `await` entre cada envio

Além de enviar as mensagens, precisamos recebê-las. Neste caso, como sabemos quantas mensagens virão, poderíamos fazer isso manualmente chamando `rx.recv().await` quatro vezes. No mundo real, porém, normalmente esperamos uma quantidade desconhecida de mensagens, então precisamos continuar aguardando até descobrir que não há mais nenhuma.

Na Listagem 16-10, usamos um loop `for` para processar todos os itens recebidos de um channel síncrono. Rust ainda não tem uma forma de usar `for` diretamente com uma sequência de itens produzida de modo assíncrono; por isso, usamos um tipo de loop que ainda não vimos: o loop condicional `while let`. Ele é a versão em loop da construção `if let` que vimos na seção "Fluxo de controle conciso com `if let` e `let...else`", no Capítulo 6. O loop continua executando enquanto o padrão especificado continuar casando com o valor.

A chamada `rx.recv` produz uma future, que aguardamos com `await`. O runtime pausa essa future até que ela esteja pronta. Quando uma mensagem chega, a future resolve para `Some(message)`, uma vez para cada mensagem recebida. Quando o channel fecha, independentemente de alguma mensagem ter chegado ou não, a future resolve para `None`, indicando que não há mais valores e que devemos parar de fazer _polling_, isto é, parar de aguardar.

O loop `while let` junta tudo isso. Se o resultado de `rx.recv().await` for `Some(message)`, temos acesso à mensagem e podemos usá-la no corpo do loop, assim como faríamos com `if let`. Se o resultado for `None`, o loop termina. Ao fim de cada iteração, o código chega novamente ao ponto de `await`, e o runtime pausa a future até outra mensagem chegar.

Agora o código envia e recebe todas as mensagens. Infelizmente, ainda há dois problemas. Primeiro, as mensagens não chegam em intervalos de meio segundo. Elas chegam todas de uma vez, 2 segundos (2.000 milissegundos) depois do início do programa. Segundo, o programa também nunca termina: ele fica esperando novas mensagens para sempre. Você precisará encerrá-lo com <kbd>Ctrl</kbd>+<kbd>C</kbd>.

### Código dentro de um bloco async executa de forma linear

Vamos começar entendendo por que as mensagens chegam todas juntas depois do atraso completo, em vez de aparecerem com uma pausa entre uma e outra. Dentro de um mesmo bloco `async`, a ordem em que as palavras-chave `await` aparecem no código também é a ordem em que elas são executadas quando o programa roda.

Na Listagem 17-10 há apenas um bloco `async`, então tudo nele é executado de forma linear. Ainda não existe concorrência. Todas as chamadas `tx.send` acontecem, intercaladas com todas as chamadas `trpl::sleep` e seus respectivos pontos de `await`. Só depois disso o loop `while let` chega a qualquer ponto de `await` nas chamadas `recv`.

Para obter o comportamento desejado, em que o atraso de `sleep` acontece entre uma mensagem e outra, precisamos colocar as operações de `tx` e `rx` em seus próprios blocos `async`, como na Listagem 17-11. Assim, o runtime pode executar cada bloco separadamente usando `trpl::join`, como na Listagem 17-8. Mais uma vez, aguardamos o resultado de `trpl::join`, e não as futures individuais. Se aguardássemos as futures uma depois da outra, voltaríamos a um fluxo sequencial, justamente o que queremos evitar.

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

<a id="listagem-17-11"></a>

[Listagem 17-11](#listagem-17-11): Separando `send` e `recv` em blocos `async` próprios e aguardando as futures desses blocos

Com o código atualizado da Listagem 17-11, as mensagens são impressas em intervalos de 500 milissegundos, em vez de aparecerem todas de uma vez depois de 2 segundos.

### Movendo ownership para um bloco async

O programa ainda não termina, porém, por causa da interação entre o loop `while let` e `trpl::join`:

- A future retornada por `trpl::join` só termina quando as duas futures passadas para ela terminam.
- A future `tx_fut` termina quando acaba de dormir depois de enviar a última mensagem de `vals`.
- A future `rx_fut` não termina até o loop `while let` terminar.
- O loop `while let` não termina até `rx.recv` retornar `None`.
- Aguardar `rx.recv` só retorna `None` quando a outra ponta do channel é fechada.
- O channel só é fechado se chamarmos `rx.close` ou quando o lado de envio, `tx`, é descartado.
- Não chamamos `rx.close` em lugar nenhum, e `tx` só será descartado quando o bloco async mais externo, passado para `trpl::block_on`, terminar.
- Esse bloco não pode terminar porque está esperando `trpl::join` completar, o que nos leva de volta ao primeiro item desta lista.

No momento, o bloco async em que enviamos as mensagens apenas empresta `tx`, porque enviar uma mensagem não exige ownership. Mas, se pudéssemos mover `tx` para dentro desse bloco async, ele seria descartado quando o bloco terminasse. Na seção "Capturando referências ou movendo ownership", do Capítulo 13, você aprendeu a usar a palavra-chave `move` com closures; e, como discutimos na seção "Usando closures `move` com threads", do Capítulo 16, muitas vezes precisamos mover dados para dentro de closures ao trabalhar com threads. A mesma dinâmica básica vale para blocos async, então `move` funciona com blocos `async` da mesma forma que funciona com closures.

Na Listagem 17-12, mudamos o bloco usado para enviar mensagens de `async` para `async move`.

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

<a id="listagem-17-12"></a>

[Listagem 17-12](#listagem-17-12): Uma revisão do código da Listagem 17-11 que encerra corretamente ao terminar

Quando executamos esta versão, o programa encerra de forma limpa depois que a última mensagem é enviada e recebida. A seguir, vamos ver o que precisa mudar para enviar dados a partir de mais de uma future.

### Juntando várias futures com a macro `join!`

Esse channel async também é um channel de vários produtores, então podemos chamar `clone` em `tx` se quisermos enviar mensagens a partir de várias futures, como mostra a Listagem 17-13.

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

<a id="listagem-17-13"></a>

[Listagem 17-13](#listagem-17-13): Usando vários produtores com blocos async

Primeiro, clonamos `tx`, criando `tx1` fora do primeiro bloco async. Movemos `tx1` para dentro desse bloco, assim como fizemos antes com `tx`. Depois, movemos o `tx` original para um novo bloco async, onde enviamos mais mensagens com um atraso um pouco maior. Por acaso colocamos esse novo bloco async depois do bloco que recebe mensagens, mas ele poderia vir antes sem problema. O que importa é a ordem em que as futures são aguardadas, não a ordem em que elas são criadas.

Os dois blocos async que enviam mensagens precisam ser blocos `async move`, para que tanto `tx` quanto `tx1` sejam descartados quando esses blocos terminarem. Caso contrário, voltaríamos ao mesmo loop infinito de antes.

Por fim, trocamos `trpl::join` por `trpl::join!` para lidar com a future adicional: a macro `join!` aguarda um número arbitrário de futures quando sabemos esse número em tempo de compilação. Mais adiante neste capítulo, discutiremos como aguardar uma coleção de futures cujo tamanho não conhecemos de antemão.

Agora vemos todas as mensagens das duas futures de envio. Como essas futures usam atrasos ligeiramente diferentes depois de enviar, as mensagens também são recebidas nesses intervalos diferentes:

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

Exploramos como usar passagem de mensagens para enviar dados entre futures, como o código dentro de um bloco async executa em sequência, como mover ownership para dentro de um bloco async e como juntar várias futures. Agora, vamos discutir como e por que avisar ao runtime que ele pode trocar para outra tarefa.
