---
title: "Um olhar mais de perto nas traits para async"
chapter_code: 17-05
slug: um-olhar-mais-de-perto-nas-traits-para-async
---

# Um olhar mais de perto nas traits para async

Ao longo deste capítulo, usamos as traits `Future`, `Stream` e `StreamExt` de várias maneiras. Até agora, porém, evitamos entrar demais nos detalhes de como elas funcionam ou de como se encaixam, e isso basta na maior parte do trabalho diário com Rust. Às vezes, entretanto, você encontrará situações em que precisará entender um pouco melhor essas traits, além do tipo `Pin` e da trait `Unpin`. Nesta seção, vamos nos aprofundar o suficiente para ajudar nesses casos, deixando o mergulho _realmente_ profundo para outras documentações.

## A trait `Future`

Vamos começar olhando com mais atenção para a trait `Future`. Esta é a definição dela em Rust:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Essa definição inclui vários tipos novos e também uma sintaxe que ainda não vimos, então vamos analisá-la por partes.

Primeiro, o tipo associado `Output`, de `Future`, indica para qual valor a future resolve. Ele é análogo ao tipo associado `Item` da trait `Iterator`. Segundo, `Future` tem o método `poll`, que recebe uma referência especial `Pin` como parâmetro `self`, uma referência mutável para um tipo `Context` e retorna um `Poll<Self::Output>`. Falaremos mais sobre `Pin` e `Context` daqui a pouco. Por enquanto, vamos nos concentrar no retorno do método, o tipo `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

O tipo `Poll` lembra `Option`: ele tem uma variante com valor, `Ready(T)`, e uma sem valor, `Pending`. Mas o significado é bem diferente. A variante `Pending` indica que a future ainda tem trabalho a fazer, então quem chamou precisará verificar novamente mais tarde. A variante `Ready` indica que a `Future` terminou seu trabalho e que o valor `T` está disponível.

> **Nota:** É raro precisar chamar `poll` diretamente. Se você precisar, lembre-se de que, para a maioria das futures, quem chama não deve chamar `poll` de novo depois que a future retorna `Ready`. Muitas futures entram em pânico se forem consultadas novamente depois de ficarem prontas. Futures que podem ser consultadas de novo com segurança dizem isso explicitamente na documentação. Isso é parecido com o comportamento de `Iterator::next`.

Quando você vê código usando `await`, o Rust o compila, por baixo dos panos, para código que chama `poll`. Se voltarmos à Listagem 17-4, em que imprimimos o título de uma página para uma única URL depois que ele ficou disponível, o Rust compila aquilo para algo parecido, embora não exatamente igual, com isto:

```rust
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // Mas o que entra aqui?
    }
}
```

O que devemos fazer quando a future ainda está `Pending`? Precisamos de alguma forma de tentar de novo, e de novo, e de novo, até que a future finalmente esteja pronta. Em outras palavras, precisaríamos de um loop:

```rust
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match value {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continuar
        }
    }
}
```

Se o Rust compilasse `await` exatamente para esse código, porém, cada `await` seria bloqueante, justamente o contrário do que queremos! Em vez disso, o Rust garante que o loop possa entregar o controle para algo capaz de pausar o trabalho dessa future, executar outras futures e verificar esta novamente mais tarde. Como vimos, esse algo é um runtime async, e esse trabalho de escalonamento e coordenação é uma das principais responsabilidades dele.

Na seção ["Enviando dados entre duas tarefas com passagem de mensagens"](/livro/cap17-02-aplicando-concorrencia-com-async#enviando-dados-entre-duas-tarefas-com-passagem-de-mensagens), descrevemos a espera em `rx.recv`. A chamada `recv` retorna uma future, e aguardar essa future faz _poll_ nela. Observamos que o runtime pausa a future até que ela esteja pronta com `Some(message)` ou com `None`, quando o channel fecha. Com uma compreensão mais detalhada da trait `Future`, especialmente de `Future::poll`, podemos ver como isso funciona. O runtime sabe que a future ainda não está pronta quando ela retorna `Poll::Pending`. Do mesmo modo, o runtime sabe que a future _está_ pronta e pode avançar quando `poll` retorna `Poll::Ready(Some(message))` ou `Poll::Ready(None)`.

Os detalhes exatos de como um runtime faz isso ficam fora do escopo deste livro, mas a ideia central é esta: o runtime faz _poll_ em cada future sob sua responsabilidade e coloca a future em espera novamente quando ela ainda não está pronta.

## O tipo `Pin` e a trait `Unpin`

Na Listagem 17-13, usamos a macro `trpl::join!` para aguardar três futures. Porém, é comum termos uma coleção, como um vetor, contendo um número de futures que só será conhecido em tempo de execução. Vamos alterar a Listagem 17-13 para o código da Listagem 17-23, colocando as três futures em um vetor e chamando `trpl::join_all`, que ainda não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let futures: Vec<Box<dyn Future<Output = ()>>> =
    vec![Box::new(tx1_fut), Box::new(rx_fut), Box::new(tx_fut)];

trpl::join_all(futures).await;
```

<a id="listagem-17-23"></a>

[Listagem 17-23](#listagem-17-23): Aguardando futures em uma coleção

Colocamos cada future dentro de um `Box` para transformá-las em _trait objects_, assim como fizemos na seção "Retornando erros de `run`", no Capítulo 12. Vamos estudar trait objects em detalhe no Capítulo 18. Usar trait objects nos permite tratar as futures anônimas produzidas por esses blocos como se fossem do mesmo tipo, porque todas implementam a trait `Future`.

Isso pode surpreender. Afinal, nenhum dos blocos async retorna nada, então cada um produz uma `Future<Output = ()>`. Mas lembre-se: `Future` é uma trait, e o compilador cria um enum único para cada bloco async, mesmo quando eles têm tipos de saída idênticos. Do mesmo modo que não podemos colocar duas structs diferentes escritas à mão em um `Vec` sem algum tipo comum, também não podemos misturar enums diferentes gerados pelo compilador.

Depois, passamos a coleção de futures para `trpl::join_all` e aguardamos o resultado. O código, porém, não compila. Esta é a parte relevante da mensagem de erro:

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
```

A observação nessa mensagem diz que devemos usar a macro `pin!` para _fixar_ os valores. Fixar um valor significa colocá-lo dentro do tipo `Pin`, que garante que esse valor não será movido na memória. A mensagem de erro diz que isso é necessário porque `dyn Future<Output = ()>` precisa implementar a trait `Unpin`, mas não implementa.

A função `trpl::join_all` retorna uma struct chamada `JoinAll`. Essa struct é genérica sobre um tipo `F`, que precisa implementar a trait `Future`. Quando aguardamos uma future diretamente com `await`, a future é fixada implicitamente. Por isso, não precisamos usar `pin!` em todo lugar em que queremos aguardar futures.

Aqui, porém, não estamos aguardando uma future diretamente. Em vez disso, construímos uma nova future, `JoinAll`, passando uma coleção de futures para `join_all`. A assinatura de `join_all` exige que os tipos dos itens na coleção implementem `Future`, e `Box<T>` só implementa `Future` se o `T` que ele envolve for uma future que também implementa `Unpin`.

Isso é muita coisa para absorver! Para entender de verdade, vamos aprofundar um pouco mais como a trait `Future` funciona, especialmente em relação a pinning. Veja de novo a definição da trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

O parâmetro `cx` e o tipo `Context` são a chave para o runtime saber quando deve verificar uma determinada future sem deixar de ser preguiçoso. Mais uma vez, os detalhes disso ficam fora do escopo deste capítulo, e você normalmente só precisa pensar neles ao escrever uma implementação personalizada de `Future`. Vamos nos concentrar no tipo de `self`, porque esta é a primeira vez que vemos um método em que `self` tem uma anotação de tipo. Uma anotação de tipo para `self` funciona como anotações em outros parâmetros de função, mas com duas diferenças importantes:

- Ela diz ao Rust qual deve ser o tipo de `self` para que o método possa ser chamado.
- Ela não pode ser qualquer tipo. Está restrita ao tipo em que o método foi implementado, a uma referência ou smart pointer para esse tipo, ou a um `Pin` envolvendo uma referência para esse tipo.

Veremos mais sobre essa sintaxe no [Capítulo 18](/livro/cap18-00-recursos-de-oop). Por enquanto, basta saber que, se quisermos fazer _poll_ em uma future para verificar se ela está `Pending` ou `Ready(Output)`, precisamos de uma referência mutável envolvida por `Pin` para esse tipo.

`Pin` é um envoltório para tipos parecidos com ponteiros, como `&`, `&mut`, `Box` e `Rc`. Tecnicamente, `Pin` funciona com tipos que implementam as traits `Deref` ou `DerefMut`, mas na prática isso equivale a trabalhar com referências e smart pointers. `Pin` não é um ponteiro em si e não tem um comportamento próprio como `Rc` e `Arc` têm com contagem de referências; ele é apenas uma ferramenta que o compilador pode usar para impor restrições ao uso de ponteiros.

Lembrar que `await` é implementado em termos de chamadas a `poll` começa a explicar a mensagem de erro que vimos antes, mas a mensagem falava de `Unpin`, não de `Pin`. Então qual é exatamente a relação entre `Pin` e `Unpin`, e por que `Future` precisa que `self` esteja em um tipo `Pin` para chamar `poll`?

Lembre-se de que, como vimos antes neste capítulo, uma sequência de pontos de `await` em uma future é compilada para uma máquina de estados. O compilador garante que essa máquina de estados siga todas as regras normais de segurança de Rust, incluindo borrowing e ownership. Para isso, o Rust analisa quais dados são necessários entre um ponto de `await` e o próximo ponto de `await`, ou o fim do bloco async. Em seguida, cria uma variante correspondente na máquina de estados compilada. Cada variante recebe o acesso necessário aos dados usados naquele trecho do código-fonte, seja assumindo ownership desses dados, seja obtendo uma referência mutável ou imutável para eles.

Até aqui, tudo bem: se errarmos alguma regra de ownership ou referências dentro de um bloco async, o borrow checker avisará. Quando queremos mover a future correspondente a esse bloco, como ao colocá-la em um `Vec` para passá-la a `join_all`, a situação fica mais delicada.

Mover uma future, seja inserindo-a em uma estrutura de dados para usá-la com `join_all`, seja retornando-a de uma função, significa mover a máquina de estados que o Rust criou para nós. Ao contrário da maioria dos outros tipos em Rust, as futures criadas para blocos async podem acabar tendo referências para si mesmas nos campos de alguma variante, como mostra a ilustração simplificada da Figura 17-4.

![Tabela representando uma future fut1 com valores 0 e 1 e uma seta da terceira linha de volta para a segunda, indicando uma referência interna.](https://doc.rust-lang.org/book/img/trpl17-04.svg)

*Figura 17-4: Um tipo de dados autorreferencial*

Por padrão, qualquer objeto que tenha uma referência para si mesmo é inseguro de mover, porque referências sempre apontam para o endereço real de memória daquilo a que se referem, como na Figura 17-5. Se você mover a estrutura de dados, essas referências internas continuarão apontando para o local antigo. Esse local agora é inválido: seu valor não será atualizado quando você alterar a estrutura, e, mais importante, o computador fica livre para reutilizar aquela memória para outra finalidade. Você poderia acabar lendo dados completamente sem relação com o objeto original.

![Uma future fut1 invalidada e uma future fut2 com ponteiro para o local antigo de fut1.](https://doc.rust-lang.org/book/img/trpl17-05.svg)

*Figura 17-5: O resultado inseguro de mover um tipo de dados autorreferencial*

Em teoria, o compilador Rust poderia tentar atualizar todas as referências para um objeto sempre que ele fosse movido, mas isso poderia adicionar bastante custo de desempenho, especialmente se uma rede inteira de referências precisasse ser atualizada. Se, em vez disso, pudermos garantir que a estrutura de dados em questão _não se move na memória_, não precisamos atualizar nenhuma referência. É exatamente para isso que o borrow checker de Rust existe: em código seguro, ele impede que você mova qualquer item enquanto há uma referência ativa para ele.

`Pin` se apoia nessa ideia para nos dar a garantia exata de que precisamos. Quando _fixamos_ um valor envolvendo um ponteiro para esse valor em `Pin`, ele não pode mais ser movido. Assim, se você tem `Pin<Box<SomeType>>`, na verdade fixa o valor `SomeType`, _não_ o ponteiro `Box`. A Figura 17-6 ilustra esse processo.

![Um Pin apontando por meio de Box para uma future autorreferencial fixada.](https://doc.rust-lang.org/book/img/trpl17-06.svg)

*Figura 17-6: Fixando um `Box` que aponta para uma future autorreferencial*

Na verdade, o ponteiro `Box` ainda pode se mover livremente. Lembre-se: o que importa é garantir que os dados finais referenciados permaneçam no mesmo lugar. Se um ponteiro se move, mas _os dados para os quais ele aponta_ continuam no mesmo lugar, como na Figura 17-7, não há problema potencial. O ponto principal é que o tipo autorreferencial em si não pode se mover, porque ele continua fixado.

![Um Pin agora apontando por b2 em vez de b1, com os dados fixados permanecendo no mesmo lugar.](https://doc.rust-lang.org/book/img/trpl17-07.svg)

*Figura 17-7: Movendo um `Box` que aponta para uma future autorreferencial fixada*

No entanto, a maioria dos tipos é perfeitamente segura de mover, mesmo quando está atrás de um ponteiro `Pin`. Só precisamos pensar em pinning quando os itens têm referências internas. Valores primitivos, como números e booleanos, são seguros porque obviamente não têm referências internas. O mesmo vale para a maioria dos tipos que você costuma usar em Rust. Você pode mover um `Vec`, por exemplo, sem se preocupar.

Precisamos de uma forma de dizer ao compilador que não há problema em mover valores nesses casos. É aí que entra `Unpin`.

`Unpin` é uma trait marcadora, parecida com as traits `Send` e `Sync` que vimos no Capítulo 16, e por isso não tem funcionalidade própria. Traits marcadoras existem apenas para informar ao compilador que é seguro usar um tipo em determinado contexto. `Unpin` informa ao compilador que um tipo específico _não_ precisa manter garantias especiais sobre poder ou não ser movido com segurança.

Assim como acontece com `Send` e `Sync`, o compilador implementa `Unpin` automaticamente para todos os tipos em que consegue provar que isso é seguro. O caso especial, também semelhante a `Send` e `Sync`, ocorre quando `Unpin` _não_ é implementado para um tipo. A notação para isso é `impl !Unpin for SomeType`, em que `SomeType` é o nome de um tipo que _precisa_ manter essas garantias para ser seguro sempre que um ponteiro para ele é usado em um `Pin`.

Em outras palavras, há duas ideias importantes na relação entre `Pin` e `Unpin`. Primeiro, `Unpin` é o caso "normal", e `!Unpin` é o caso especial. Segundo, o fato de um tipo implementar `Unpin` ou `!Unpin` _só_ importa quando estamos usando um ponteiro fixado para esse tipo, como `Pin<&mut SomeType>`.

Para deixar isso mais concreto, pense em uma `String`: ela tem um tamanho e os caracteres Unicode que a compõem. Podemos envolver uma `String` em `Pin`, como mostra a Figura 17-8. Porém, `String` implementa `Unpin` automaticamente, assim como a maioria dos outros tipos em Rust.

![Um Pin apontando para uma String hello; a borda tracejada indica que String implementa Unpin.](https://doc.rust-lang.org/book/img/trpl17-08.svg)

*Figura 17-8: Fixando uma `String`; a linha tracejada indica que `String` implementa `Unpin` e, portanto, não está realmente presa*

Como resultado, podemos fazer coisas que seriam ilegais se `String` implementasse `!Unpin`, como substituir uma string por outra exatamente no mesmo local da memória, como na Figura 17-9. Isso não viola o contrato de `Pin`, porque `String` não tem referências internas que a tornem insegura de mover. É justamente por isso que ela implementa `Unpin`, e não `!Unpin`.

![Um Pin apontando de s1 para s2 goodbye.](https://doc.rust-lang.org/book/img/trpl17-09.svg)

*Figura 17-9: Substituindo uma `String` por outra completamente diferente na memória*

Agora sabemos o suficiente para entender os erros daquela chamada a `join_all` da Listagem 17-23. Inicialmente, tentamos mover as futures produzidas por blocos async para dentro de um `Vec<Box<dyn Future<Output = ()>>>`, mas, como vimos, essas futures podem ter referências internas e por isso não implementam `Unpin` automaticamente. Depois de fixá-las, podemos passar o tipo `Pin` resultante para o `Vec`, confiantes de que os dados internos das futures _não_ serão movidos. A Listagem 17-24 mostra como corrigir o código chamando a macro `pin!` onde cada uma das três futures é definida e ajustando o tipo do trait object.

**Arquivo: src/main.rs**

```rust
use std::pin::{Pin, pin};

let tx1_fut = pin!(async move {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("future"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        trpl::sleep(Duration::from_secs(1)).await;
    }
});

let rx_fut = pin!(async {
    while let Some(value) = rx.recv().await {
        println!("received '{value}'");
    }
});

let tx_fut = pin!(async move {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        trpl::sleep(Duration::from_secs(1)).await;
    }
});

let futures: Vec<Pin<&mut dyn Future<Output = ()>>> =
    vec![tx1_fut, rx_fut, tx_fut];

trpl::join_all(futures).await;
```

<a id="listagem-17-24"></a>

[Listagem 17-24](#listagem-17-24): Fixando futures para movê-las para o vetor

Esse exemplo agora compila e roda, e poderíamos adicionar ou remover futures do vetor em tempo de execução e aguardá-las todas juntas.

`Pin` e `Unpin` são importantes principalmente para construir bibliotecas de baixo nível ou para implementar um runtime, mais do que para código Rust do dia a dia. Ainda assim, quando essas traits aparecerem em mensagens de erro, agora você terá uma ideia melhor de como corrigir seu código.

> **Nota:** A combinação de `Pin` e `Unpin` permite implementar com segurança toda uma classe de tipos complexos em Rust que seriam difíceis justamente por serem autorreferenciais. Tipos que exigem `Pin` aparecem com mais frequência em Rust async hoje, mas, de vez em quando, você também pode encontrá-los em outros contextos.
>
> Os detalhes de como `Pin` e `Unpin` funcionam, e das regras que eles precisam manter, são explicados extensamente na documentação da API de [`std::pin`](https://doc.rust-lang.org/std/pin/index.html). Se quiser aprender mais, esse é um ótimo ponto de partida.
>
> Para entender ainda melhor como tudo funciona por baixo dos panos, veja os capítulos [2](https://rust-lang.github.io/async-book/02_execution/01_chapter.html) e [4](https://rust-lang.github.io/async-book/04_pinning/01_chapter.html) de [_Asynchronous Programming in Rust_](https://rust-lang.github.io/async-book/).

## A trait `Stream`

Agora que você tem uma noção mais profunda das traits `Future`, `Pin` e `Unpin`, podemos voltar nossa atenção para a trait `Stream`. Como você aprendeu antes neste capítulo, streams são parecidos com iterators assíncronos. Diferentemente de `Iterator` e `Future`, porém, `Stream` ainda não tem uma definição na biblioteca padrão no momento em que este texto foi escrito, embora exista uma definição muito comum no crate `futures`, usada em todo o ecossistema.

Vamos revisar as definições de `Iterator` e `Future` antes de ver como uma trait `Stream` pode juntar as duas ideias. De `Iterator`, temos a ideia de uma sequência: seu método `next` fornece um `Option<Item>`. De `Future`, temos a ideia de prontidão ao longo do tempo: seu método `poll` fornece um `Poll<Output>`. Para representar uma sequência de itens que ficam prontos ao longo do tempo, definimos uma trait `Stream` que reúne essas características:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

A trait `Stream` define um tipo associado chamado `Item`, que indica o tipo dos itens produzidos pelo stream. Isso é parecido com `Iterator`, em que pode haver de zero a muitos itens, e diferente de `Future`, em que sempre há um único `Output`, mesmo que esse output seja o tipo unitário `()`.

`Stream` também define um método para obter esses itens. Chamamos esse método de `poll_next` para deixar claro que ele faz _poll_ da mesma forma que `Future::poll` e produz uma sequência de itens da mesma forma que `Iterator::next`. O tipo de retorno combina `Poll` com `Option`. O tipo externo é `Poll`, porque precisamos verificar prontidão, como em uma future. O tipo interno é `Option`, porque precisamos sinalizar se ainda existem mais mensagens, como em um iterator.

Algo muito parecido com essa definição provavelmente acabará fazendo parte da biblioteca padrão de Rust. Enquanto isso, ela faz parte do conjunto de ferramentas da maioria dos runtimes, então você pode contar com esse padrão, e tudo que discutimos aqui costuma se aplicar.

Nos exemplos da seção ["Streams: futures em sequência"](/livro/cap17-04-streams-futures-em-sequencia), porém, não usamos `poll_next` nem `Stream` diretamente. Em vez disso, usamos `next` e `StreamExt`. Poderíamos trabalhar diretamente com a API `poll_next`, escrevendo à mão nossas próprias máquinas de estado para `Stream`, assim como _poderíamos_ trabalhar com futures diretamente por meio do método `poll`. Mas usar `await` é muito mais agradável, e a trait `StreamExt` fornece o método `next` para permitir isso:

```rust
trait StreamExt: Stream {
    async fn next(&mut self) -> Option<Self::Item>
    where
        Self: Unpin;

    // outros métodos...
}
```

> **Nota:** A definição real que usamos antes neste capítulo é um pouco diferente, porque dá suporte a versões de Rust que ainda não aceitavam funções async em traits. Por isso, ela se parece com isto:
>
> ```rust
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> O tipo `Next` é uma `struct` que implementa `Future` e permite nomear o tempo de vida da referência a `self` com `Next<'_, Self>`, para que `await` possa funcionar com esse método.

A trait `StreamExt` também é onde ficam todos os métodos interessantes disponíveis para streams. `StreamExt` é implementada automaticamente para todo tipo que implementa `Stream`, mas as duas traits são definidas separadamente para permitir que a comunidade evolua as APIs de conveniência sem afetar a trait fundamental.

Na versão de `StreamExt` usada pelo crate `trpl`, a trait não apenas define o método `next`, como também fornece uma implementação padrão de `next` que lida corretamente com os detalhes de chamar `Stream::poll_next`. Isso significa que, mesmo quando você precisar escrever seu próprio tipo de dado em streaming, você _só_ precisará implementar `Stream`; quem usar seu tipo poderá usar `StreamExt` e seus métodos automaticamente.

Isso é tudo que vamos cobrir sobre os detalhes de mais baixo nível dessas traits. Para concluir, vamos pensar em como futures, incluindo streams, tarefas e threads se encaixam.
