---
title: "Um olhar mais de perto nas traits para async"
chapter_code: 17-05
slug: um-olhar-mais-de-perto-nas-traits-para-async
---

# Um Olhar Mais de Perto nas Traits para Async

Ao longo do capítulo usamos `Future`, `Stream` e `StreamExt` de várias formas. Evitamos detalhes de implementação — suficiente para o dia a dia. Às vezes você precisa entender `Pin` e `Unpin`; esta seção cobre o essencial. O mergulho profundo fica para outra documentação.

## A trait `Future`

Definição em Rust:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`Output` é o que a future resolve (como `Item` em `Iterator`). `poll` recebe `Pin<&mut Self>` e `Context`, e retorna `Poll<Self::Output>`.

`Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Parecido com `Option`, mas `Pending` significa que a future ainda tem trabalho; o chamador verifica de novo depois. `Ready(T)` significa conclusão.

> **Nota:** É raro chamar `poll` diretamente. Depois de `Ready`, não chame `poll` de novo na maioria das futures (muitas entram em pânico). Futures seguras para poll de novo dizem isso na documentação — como `Iterator::next`.

`await` compila para chamadas a `poll`. A Listagem 17-4, ao imprimir o título de uma URL, vira algo como (não exatamente):

```rust
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // Mas o que colocar aqui?
    }
}
```

Com `Pending`, precisamos tentar de novo em loop até `Ready`:

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

Se fosse exatamente isso, cada `await` bloquearia — o oposto do objetivo. O loop precisa ceder controle a algo que pause esta future, rode outras e volte depois: o runtime async.

Em Enviando dados entre duas tarefas com passagem de mensagens, `rx.recv` retorna future; `await` faz poll. O runtime pausa até `Some(message)` ou `None`. Com `Future::poll`: `Poll::Pending` = não pronto; `Poll::Ready(Some(...))` ou `Poll::Ready(None)` = pronto para avançar.

Os detalhes do runtime estão além deste livro; o essencial: o runtime _poll_ cada future sob sua responsabilidade e a recoloca quando não está pronta.

## O tipo `Pin` e a trait `Unpin`

Na Listagem 17-13 usamos `join!` para três futures. É comum ter um `Vec` de futures com tamanho só conhecido em runtime. Mudemos a Listagem 17-13 para colocar três futures num `Vec` e chamar `trpl::join_all` (Listagem 17-23) — que ainda não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
        let futures: Vec<Box<dyn Future<Output = ()>>> =
            vec![Box::new(tx1_fut), Box::new(rx_fut), Box::new(tx_fut)];

        trpl::join_all(futures).await;
```

<a id="listagem-17-23"></a>

[Listagem 17-23](#listagem-17-23): Aguardando futures numa coleção

Cada future em `Box` como _trait object_ (como em Retornar erros de `run` no Capítulo 12; trait objects no Capítulo 18). Tipos anônimos de cada bloco `async` são enums diferentes do compilador, mesmo com `Output = ()`.

Erro relevante:

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

Use a macro `pin!` para _fixar_ (_pin_) valores: `Pin` garante que não se movem na memória. `dyn Future<Output = ()>` precisa de `Unpin`, que não tem.

`join_all` retorna `JoinAll<F>` com `F: Future`. `await` direto fixa a future implicitamente. Aqui construímos `JoinAll` passando a coleção; `Box<T>` implementa `Future` só se `T` for future `Unpin`.

`Future::poll` exige `self: Pin<&mut Self>`. `Pin` envolve referências e smart pointers; não é ponteiro nem conta referências — ferramenta do compilador.

Await vira `poll`; o erro falava em `Unpin`. Pontos de await viram máquina de estados; o compilador garante borrowing. Ao _mover_ a future (ex.: para `Vec` em `join_all`), move-se a máquina de estados. Futures de blocos `async` podem ter referências internas a si mesmas (Figura 17-4).

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-04.svg" class="center" alt="Tabela representando future fut1 com valores 0 e 1 e seta da terceira linha de volta à segunda, referência interna." />

<figcaption>Figura 17-4: Um tipo de dados autorreferencial</figcaption>

</figure>

Por padrão, objeto com referência a si mesmo é inseguro mover: referências apontam para endereços reais (Figura 17-5). Mover a estrutura deixa referências internas inválidas — memória pode ser reutilizada.

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-05.svg" class="center" alt="fut1 invalidado e fut2 com ponteiro para local antigo de fut1." />

<figcaption>Figura 17-5: Resultado inseguro de mover tipo autorreferencial</figcaption>

</figure>

Atualizar todas as referências ao mover seria caro. Garantir que a estrutura _não se move na memória_ evita isso — o que o borrow checker já faz em código seguro.

`Pin` garante isso: `Pin<Box<SomeType>>` fixa o valor `SomeType`, não o ponteiro `Box` (Figura 17-6).

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-06.svg" class="center" alt="Pin apontando via Box para future fut autorreferencial fixada." />

<figcaption>Figura 17-6: Fixar um `Box` que aponta para future autorreferencial</figcaption>

</figure>

O `Box` pode se mover; o que importa é o destino dos dados (Figura 17-7). O tipo autorreferencial em si não pode se mover enquanto estiver fixado.

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-07.svg" class="center" alt="Pin agora via b2 em vez de b1; dados em pinned inalterados." />

<figcaption>Figura 17-7: Mover o `Box` que aponta para future autorreferencial fixada</figcaption>

</figure>

A maioria dos tipos pode mover com segurança mesmo atrás de `Pin`. Só precisamos de pinning com referências internas. Primitivos não têm. `Vec` move livremente. `String` implementa `Unpin` (Figura 17-8).

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-08.svg" class="center" alt="Pin apontando para String hello; borda tracejada indica Unpin." />

<figcaption>Figura 17-8: Fixar `String`; linha tracejada indica `Unpin` — não está realmente fixada</figcaption>

</figure>

Com `Unpin`, podemos substituir o conteúdo no mesmo endereço (Figura 17-9) sem violar o contrato de `Pin`.

<figure>

<img src="https://doc.rust-lang.org/book/img/trpl17-09.svg" class="center" alt="Pin apontando de s1 para s2 goodbye." />

<figcaption>Figura 17-9: Substituir `String` por outra no mesmo lugar na memória</figcaption>

</figure>

`Unpin` é trait marcador (como `Send`/`Sync` no Capítulo 16): sem métodos; informa que o tipo _não_ precisa das garantias especiais de pinning. `Unpin` é o caso normal; `!Unpin` é o especial. Só importa com ponteiro fixado como `Pin<&mut SomeType>`.

Correção na Listagem 17-24: `pin!` em cada future e tipo `Vec<Pin<&mut dyn Future<Output = ()>>>`.

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

Compila e roda; podemos juntar número variável de futures em runtime.

`Pin` e `Unpin` importam mais para bibliotecas de baixo nível e runtimes. Em erros do dia a dia, você já tem uma pista de como corrigir.

> **Nota:** `Pin`/`Unpin` permitem tipos autorreferenciais seguros em Rust. Aparecem sobretudo em async, mas também noutros contextos. Detalhes em [`std::pin`](https://doc.rust-lang.org/std/pin/index.html) e nos capítulos [2](https://rust-lang.github.io/async-book/02_execution/01_chapter.html) e [4](https://rust-lang.github.io/async-book/04_pinning/01_chapter.html) de [_Asynchronous Programming in Rust_](https://rust-lang.github.io/async-book/).

## A trait `Stream`

`Stream` ainda não está na biblioteca padrão; a definição comum vem do crate `futures`.

De `Iterator`: sequência via `next` → `Option<Item>`. De `Future`: prontidão no tempo via `poll` → `Poll<Output>`. `Stream` une os dois:

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

`Item` como em `Iterator` (zero a muitos itens); diferente de `Future` (um `Output`, mesmo que `()`).

`poll_next` combina `poll` e `next`; retorno `Poll<Option<Item>>`: `Poll` para prontidão, `Option` para fim da sequência.

Em Streams: futures em sequência usamos `next` e `StreamExt`, não `poll_next` direto — como `await` em vez de `poll` manual.

```rust
trait StreamExt: Stream {
    async fn next(&mut self) -> Option<Self::Item>
    where
        Self: Unpin;

    // outros métodos...
}
```

> **Nota:** A definição real no `trpl` difere um pouco em versões de Rust sem funções async em traits:
>
> ```rust
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> O tipo `Next` implementa `Future` para `await` funcionar.

`StreamExt` concentra métodos úteis; é implementada automaticamente para todo `Stream`, separada para evoluir APIs de conveniência sem mudar a trait base.

No `trpl`, `StreamExt::next` tem implementação padrão que chama `Stream::poll_next` corretamente — ao criar seu tipo de stream, implemente só `Stream` e use `StreamExt` por cima.

Para fechar: como futures (incluindo streams), tarefas e threads se encaixam.
