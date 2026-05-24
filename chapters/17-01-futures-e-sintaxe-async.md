---
title: "Futures e a sintaxe async"
chapter_code: 17-01
slug: futures-e-a-sintaxe-async
---

# Futures e a Sintaxe Async

Os elementos centrais da programação assíncrona em Rust são _futures_ e as palavras-chave `async` e `await`.

Uma _future_ é um valor que pode não estar pronto agora, mas ficará pronto em algum momento no futuro. (Esse mesmo conceito aparece em muitas linguagens, às vezes com outros nomes, como _task_ ou _promise_.) O Rust fornece a trait `Future` como bloco de construção para que operações async diferentes possam ser implementadas com estruturas de dados diferentes, mas com uma interface comum. Em Rust, futures são tipos que implementam a trait `Future`. Cada future guarda informação sobre o progresso feito e o que significa estar “pronto”.

Você pode aplicar a palavra-chave `async` a blocos e funções para indicar que podem ser interrompidos e retomados. Dentro de um bloco ou função async, você usa `await` para _aguardar uma future_ (esperar que fique pronta). Qualquer ponto em que você aguarda uma future dentro de um bloco ou função async é um possível ponto de pausa e retomada. O processo de verificar se o valor de uma future já está disponível chama-se _polling_.

Outras linguagens, como C# e JavaScript, também usam `async` e `await` para programação assíncrona. Se você conhece essas linguagens, pode notar diferenças importantes em como o Rust trata a sintaxe — com boa razão, como veremos!

Ao escrever Rust async, usamos `async` e `await` na maior parte do tempo. O Rust compila isso em código equivalente usando a trait `Future`, assim como compila loops `for` em código equivalente usando a trait `Iterator`. Como o Rust fornece `Future`, você também pode implementá-la para seus próprios tipos quando precisar. Muitas funções deste capítulo retornam tipos com implementação própria de `Future`. Voltaremos à definição da trait no fim do capítulo; por agora, isso basta para seguir em frente.

Tudo isso pode parecer abstrato, então vamos escrever nosso primeiro programa async: um pequeno _web scraper_. Passaremos duas URLs pela linha de comando, buscaremos ambas de forma concorrente e retornaremos o resultado da que terminar primeiro. O exemplo terá bastante sintaxe nova, mas explicaremos o que for preciso ao longo do caminho.

## Nosso Primeiro Programa Async

Para manter o foco do capítulo em aprender async em vez de juntar peças do ecossistema, criamos o crate `trpl` (`trpl` é abreviação de “The Rust Programming Language”). Ele reexporta os tipos, traits e funções que você precisa, principalmente dos crates [`futures`](https://crates.io/crates/futures) e [`tokio`](https://tokio.rs). O crate `futures` é um lar oficial de experimentação em Rust para código async, e foi onde a trait `Future` foi originalmente desenhada. Tokio é o runtime async mais usado em Rust hoje, especialmente em aplicações web. Há outros runtimes excelentes, e alguns podem ser mais adequados ao seu caso. Usamos o crate `tokio` por baixo do `trpl` porque é bem testado e amplamente usado.

Em alguns casos, o `trpl` também renomeia ou envolve APIs originais para você focar nos detalhes relevantes deste capítulo. Se quiser entender o que o crate faz, veja [o código-fonte](https://github.com/rust-lang/book/tree/main/packages/trpl). Você verá de qual crate cada reexport vem; deixamos comentários explicando o papel de cada um.

Crie um novo projeto binário chamado `hello-async` e adicione o crate `trpl` como dependência:

```bash
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Agora podemos usar as peças do `trpl` para escrever o primeiro programa async. Construiremos uma pequena ferramenta de linha de comando que busca duas páginas web, extrai o elemento `<title>` de cada uma e imprime o título da página que terminar o processo inteiro primeiro.

### Definindo a função `page_title`

Comecemos com uma função que recebe uma URL, faz a requisição e retorna o texto do elemento `<title>` (veja a Listagem 17-1).

**Arquivo: src/main.rs**

```rust
use trpl::Html;

async fn page_title(url: &str) -> Option<String> {
    let response = trpl::get(url).await;
    let response_text = response.text().await;
    Html::parse(&response_text)
        .select_first("title")
        .map(|title| title.inner_html())
}
```

<a id="listagem-17-1"></a>

[Listagem 17-1](#listagem-17-1): Definindo uma função async para obter o elemento título de uma página HTML

Primeiro definimos `page_title` e a marcamos com `async`. Depois usamos `trpl::get` para buscar a URL passada e `await` para aguardar a resposta. Para obter o texto da `response`, chamamos `text` e de novo `await`. Ambos os passos são assíncronos. Para `get`, precisamos esperar o servidor enviar a primeira parte da resposta (cabeçalhos HTTP, cookies etc., que podem chegar separados do corpo). Se o corpo for grande, pode demorar para chegar tudo. Como precisamos esperar a resposta _inteira_, `text` também é async.

Precisamos aguardar explicitamente essas duas futures, porque em Rust elas são _preguiçosas_ (_lazy_): não fazem nada até você pedir com `await`. (O Rust avisa no compilador se você não usar uma future.) Isso pode lembrar iterators no capítulo Processar uma série de itens com iterators: iterators não fazem nada a menos que você chame `next` — diretamente ou via `for` ou métodos como `map` que usam `next` por baixo. O mesmo vale para futures. Essa preguiça permite ao Rust evitar executar código async até ser realmente necessário.

> **Nota:** Isso difere do que vimos com `thread::spawn` em Criando uma nova thread com `spawn`, em que a closure na outra thread começa a rodar imediatamente. Também difere de muitas outras linguagens com async. Mas é importante para o Rust manter suas garantias de desempenho, como com iterators.

Com `response_text`, parseamos em `Html` com `Html::parse`. Em vez de string bruta, temos uma estrutura mais rica. Usamos `select_first` com o seletor CSS `"title"` para o primeiro `<title>`, se existir. `select_first` retorna `Option<ElementRef>`. Por fim, usamos `Option::map` para trabalhar com o item se estiver presente (também poderíamos usar `match`; `map` é mais idiomático). No corpo de `map`, chamamos `inner_html` no título. No fim, temos `Option<String>`.

Note que `await` em Rust vem _depois_ da expressão aguardada, não antes: é uma palavra-chave _postfix_. Isso pode diferir de outras linguagens, mas em Rust encadear métodos fica mais agradável. Podemos reescrever o corpo de `page_title` encadeando `trpl::get` e `text` com `await` entre eles, como na Listagem 17-2.

**Arquivo: src/main.rs**

```rust
    let response_text = trpl::get(url).await.text().await;
```

<a id="listagem-17-2"></a>

[Listagem 17-2](#listagem-17-2): Encadeando com a palavra-chave `await`

Escrevemos nossa primeira função async! Antes de chamar em `main`, vejamos o que escrevemos e o que significa.

Quando o Rust vê um _bloco_ marcado com `async`, compila em um tipo anônimo único que implementa `Future`. Quando vê uma _função_ `async`, compila em uma função não-async cujo corpo é um bloco async. O tipo de retorno de uma função async é o tipo anônimo que o compilador cria para esse bloco.

Assim, escrever `async fn` equivale a uma função que retorna uma _future_ do tipo de retorno. Para o compilador, `async fn page_title` da Listagem 17-1 é aproximadamente equivalente a:

```rust
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Percorrendo a versão transformada:

- Usa a sintaxe `impl Trait` do Capítulo 10 em Traits como parâmetros.
- O valor retornado implementa `Future` com tipo associado `Output` igual a `Option<String>`, o mesmo retorno da versão `async fn`.
- Todo o código do corpo original fica em um bloco `async move` (blocos são expressões; este é o valor retornado).
- O bloco produz `Option<String>`, alinhado ao `Output`.
- É `async move` por causa do parâmetro `url` (falaremos mais de `async` versus `async move` adiante).

Agora podemos chamar `page_title` em `main`.

### Executando uma função async com um runtime

Para começar, obtemos o título de uma única página (Listagem 17-3). Infelizmente, este código ainda não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
async fn main() {
    let args: Vec<String> = std::env::args().collect();
    let url = &args[1];
    match page_title(url).await {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
}
```

<a id="listagem-17-3"></a>

[Listagem 17-3](#listagem-17-3): Chamando `page_title` a partir de `main` com argumento da linha de comando

Seguimos o padrão de Aceitar argumentos da linha de comando do Capítulo 12. Passamos a URL para `page_title` e aguardamos o resultado. Como a future produz `Option<String>`, usamos `match` para imprimir mensagens conforme haja ou não `<title>`.

Só podemos usar `await` em funções ou blocos async, e o Rust não permite marcar `main` como `async`.

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` não pode ser `async` porque código async precisa de um _runtime_: um crate Rust que gerencia a execução de código assíncrono. `main` pode _inicializar_ um runtime, mas não é um runtime _em si_. Todo programa Rust com async tem pelo menos um lugar que configura um runtime para executar as futures.

Muitas linguagens com async embutem um runtime; Rust não. Há muitos runtimes disponíveis, cada um com tradeoffs para seu caso (servidor web de alto throughput versus microcontrolador sem heap, por exemplo). Crates de runtime costumam oferecer versões async de E/S de arquivo ou rede.

Neste capítulo usamos `block_on` do `trpl`, que recebe uma future e bloqueia a thread atual até ela completar. Por baixo, `block_on` configura um runtime com `tokio` (comportamento parecido com `block_on` de outros runtimes). Quando a future termina, `block_on` devolve o valor produzido.

Poderíamos passar a future de `page_title` direto a `block_on`, mas na maior parte dos exemplos (e do código real) faremos mais de uma chamada async; então passamos um bloco `async` e aguardamos `page_title` dentro dele, como na Listagem 17-4.

**Arquivo: src/main.rs**

```rust
fn main() {
    let args: Vec<String> = std::env::args().collect();

    trpl::block_on(async {
        let url = &args[1];
        match page_title(url).await {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
    })
}
```

<a id="listagem-17-4"></a>

[Listagem 17-4](#listagem-17-4): Aguardando um bloco async com `trpl::block_on`

Ao executar:

```text
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Temos código async funcionando! Antes de correr duas URLs, voltemos às futures.

Cada _ponto de await_ — onde usamos `await` — é onde o controle volta ao runtime. Para isso, o Rust guarda o estado do bloco async para o runtime poder fazer outro trabalho e retomar depois. É uma máquina de estados invisível, como se você tivesse escrito um enum assim em cada await:

```rust
enum PageTitleFuture<'a> {
    Initial { url: &'a str },
    GetAwaitPoint { url: &'a str },
    TextAwaitPoint { response: trpl::Response },
}
```

Escrever as transições à mão seria trabalhoso e propenso a erros; o compilador cria e gerencia essas estruturas automaticamente. Regras de borrowing e ownership continuam valendo, e o compilador verifica e dá erros úteis.

Algo precisa executar essa máquina de estados: um runtime. (Por isso você vê _executors_ ao estudar runtimes: a parte que executa o código async.)

Agora fica claro por que `main` não pode ser `async` na Listagem 17-3: algo teria de gerenciar a future retornada por `main`, mas `main` é o ponto de entrada! Chamamos `trpl::block_on` em `main` para configurar o runtime e rodar o bloco `async` até o fim.

> **Nota:** Alguns runtimes oferecem macros para `async fn main()`. Elas reescrevem para um `fn main` normal que faz o equivalente à Listagem 17-4 com `trpl::block_on`.

### Correndo duas URLs de forma concorrente

Na Listagem 17-5, chamamos `page_title` para duas URLs e corremos com `trpl::select`, que devolve a future que terminar primeiro.

**Arquivo: src/main.rs**

```rust
use trpl::{Either, Html};

fn main() {
    let args: Vec<String> = std::env::args().collect();

    trpl::block_on(async {
        let title_fut_1 = page_title(&args[1]);
        let title_fut_2 = page_title(&args[2]);

        let (url, maybe_title) =
            match trpl::select(title_fut_1, title_fut_2).await {
                Either::Left(left) => left,
                Either::Right(right) => right,
            };

        println!("{url} returned first");
        match maybe_title {
            Some(title) => println!("Its page title was: '{title}'"),
            None => println!("It had no title."),
        }
    })
}

async fn page_title(url: &str) -> (&str, Option<String>) {
    let response_text = trpl::get(url).await.text().await;
    let title = Html::parse(&response_text)
        .select_first("title")
        .map(|title| title.inner_html());
    (url, title)
}
```

<a id="listagem-17-5"></a>

[Listagem 17-5](#listagem-17-5): Chamando `page_title` para duas URLs para ver qual retorna primeiro

Guardamos as futures em `title_fut_1` e `title_fut_2` — ainda não fazem nada, porque futures são preguiçosas. Passamos as futures a `trpl::select`, que indica qual terminou primeiro.

> **Nota:** Por baixo, `trpl::select` usa `select` do crate `futures`, mais geral e complexo; por ora pulamos esses detalhes.

Qualquer future pode “vencer”, então `select` não retorna `Result`, e sim `trpl::Either`, parecido com `Result`, mas com `Left` e `Right` em vez de sucesso/erro:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

`select` retorna `Left` com a saída do primeiro argumento se ele vencer, e `Right` com a do segundo. A ordem dos argumentos importa.

Atualizamos `page_title` para retornar também a URL, para mensagens úteis se não houver `<title>`. Com isso, você tem um pequeno web scraper: escolha duas URLs e compare. Sites podem ser consistentemente mais rápidos ou variar a cada execução — a concorrência é interessante e difícil. Você aprendeu o básico de futures; agora podemos aprofundar o que mais dá para fazer com async.
