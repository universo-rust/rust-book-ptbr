---
title: "Macros"
chapter_code: 20-05
slug: macros
---

# Macros

Usamos macros como `println!` ao longo deste livro, mas não exploramos completamente o que é uma macro e como ela funciona. O termo _macro_ refere-se a uma família de recursos em Rust — macros declarativas com `macro_rules!` e três tipos de macros procedurais:

- Macros `#[derive]` personalizadas que especificam código adicionado com o atributo `derive` usado em structs e enums
- Macros semelhantes a atributos que definem atributos personalizados utilizáveis em qualquer item
- Macros semelhantes a funções que parecem chamadas de função, mas operam nos tokens especificados como argumento

Falaremos sobre cada um deles por turno, mas primeiro, vamos ver por que precisamos de macros quando já temos funções.

## A Diferença Entre Macros e Funções

Fundamentalmente, macros são uma forma de escrever código que escreve outro código, o que é conhecido como _metaprogramação_. No [Apêndice C](/livro/cap22-03-traits-derivaveis), discutimos o atributo `derive`, que gera uma implementação de várias traits para você. Também usamos as macros `println!` e `vec!` ao longo do livro. Todas essas macros _expandem_ para produzir mais código do que o código que você escreveu manualmente.

Metaprogramação é útil para reduzir a quantidade de código que você precisa escrever e manter, que também é um dos papéis das funções. Porém, macros têm alguns poderes adicionais que funções não têm.

Uma assinatura de função deve declarar o número e o tipo dos parâmetros que a função tem. Macros, por outro lado, podem receber um número variável de parâmetros: podemos chamar `println!("hello")` com um argumento ou `println!("hello {}", name)` com dois argumentos. Além disso, macros são expandidas antes do compilador interpretar o significado do código, então uma macro pode, por exemplo, implementar uma trait em um dado tipo. Uma função não pode, porque é chamada em runtime e uma trait precisa ser implementada em compile time.

A desvantagem de implementar uma macro em vez de uma função é que definições de macro são mais complexas que definições de função porque você está escrevendo código Rust que escreve código Rust. Devido a essa indireção, definições de macro são geralmente mais difíceis de ler, entender e manter do que definições de função.

Outra diferença importante entre macros e funções é que você deve definir macros ou trazê-las para o escopo _antes_ de chamá-las em um arquivo, em oposição a funções que você pode definir em qualquer lugar e chamar em qualquer lugar.

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

## Macros Declarativas para Metaprogramação Geral

A forma mais amplamente usada de macros em Rust é a _macro declarativa_. Elas também são às vezes referidas como “macros by example”, “macros `macro_rules!`” ou simplesmente “macros”. Em sua essência, macros declarativas permitem que você escreva algo semelhante a uma expressão `match` do Rust. Como discutido no Capítulo 6, expressões `match` são estruturas de controle que recebem uma expressão, comparam o valor resultante da expressão com padrões e então executam o código associado ao padrão correspondente. Macros também comparam um valor com padrões que estão associados a código particular: nesta situação, o valor é o código-fonte literal Rust passado para a macro; os padrões são comparados com a estrutura desse código-fonte; e o código associado a cada padrão, quando corresponde, substitui o código passado para a macro. Tudo isso acontece durante a compilação.

Para definir uma macro, você usa a construção `macro_rules!`. Vamos explorar como usar `macro_rules!` olhando como a macro `vec!` é definida. O Capítulo 8 cobriu como podemos usar a macro `vec!` para criar um novo vetor com valores particulares. Por exemplo, a macro a seguir cria um novo vetor contendo três inteiros:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Também poderíamos usar a macro `vec!` para fazer um vetor de dois inteiros ou um vetor de cinco fatias de string. Não conseguiríamos usar uma função para fazer o mesmo porque não saberíamos o número ou o tipo de valores antecipadamente.

A Listagem 20-35 mostra uma definição ligeiramente simplificada da macro `vec!`.

**Arquivo: src/lib.rs**

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

[Listagem 20-35](#listagem-20-35): Uma versão simplificada da definição da macro `vec!`

> **Nota:** A definição real da macro `vec!` na biblioteca padrão inclui código para pré-alocar a quantidade correta de memória antecipadamente. Esse código é uma otimização que não incluímos aqui, para tornar o exemplo mais simples.

A anotação `#[macro_export]` indica que esta macro deve ser disponibilizada sempre que o crate no qual a macro é definida for trazido para o escopo. Sem esta anotação, a macro não pode ser trazida para o escopo.

Em seguida, iniciamos a definição da macro com `macro_rules!` e o nome da macro que estamos definindo _sem_ o ponto de exclamação. O nome, neste caso `vec`, é seguido por chaves denotando o corpo da definição da macro.

A estrutura no corpo de `vec!` é semelhante à estrutura de uma expressão `match`. Aqui temos um braço com o padrão `( $( $x:expr ),* )`, seguido por `=>` e o bloco de código associado a este padrão. Se o padrão corresponder, o bloco de código associado será emitido. Dado que este é o único padrão nesta macro, há apenas uma forma válida de corresponder; qualquer outro padrão resultará em um erro. Macros mais complexas terão mais de um braço.

A sintaxe de padrão válida em definições de macro é diferente da sintaxe de padrão coberta no Capítulo 19 porque padrões de macro são comparados com a estrutura do código Rust em vez de valores. Vamos percorrer o que os pedaços do padrão na Listagem 20-35 significam; para a sintaxe completa de padrões de macro, consulte a [Referência Rust][ref].

Primeiro, usamos um conjunto de parênteses para englobar o padrão inteiro. Usamos um cifrão (`$`) para declarar uma variável no sistema de macros que conterá o código Rust correspondente ao padrão. O cifrão deixa claro que esta é uma variável de macro em oposição a uma variável Rust regular. Em seguida vem um conjunto de parênteses que captura valores que correspondem ao padrão dentro dos parênteses para uso no código de substituição. Dentro de `$()` está `$x:expr`, que corresponde a qualquer expressão Rust e dá à expressão o nome `$x`.

A vírgula seguindo `$()` indica que um caractere separador de vírgula literal deve aparecer entre cada instância do código que corresponde ao código em `$()`. O `*` especifica que o padrão corresponde a zero ou mais do que precede o `*`.

Quando chamamos esta macro com `vec![1, 2, 3];`, o padrão `$x` corresponde três vezes com as três expressões `1`, `2` e `3`.

Agora vamos olhar o padrão no corpo do código associado a este braço: `temp_vec.push()` dentro de `$()*` é gerado para cada parte que corresponde a `$()` no padrão zero ou mais vezes, dependendo de quantas vezes o padrão corresponde. O `$x` é substituído por cada expressão correspondida. Quando chamamos esta macro com `vec![1, 2, 3];`, o código gerado que substitui esta chamada de macro será o seguinte:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Definimos uma macro que pode receber qualquer número de argumentos de qualquer tipo e pode gerar código para criar um vetor contendo os elementos especificados.

Para aprender mais sobre como escrever macros, consulte a documentação online ou outros recursos, como [“The Little Book of Rust Macros”][tlborm] iniciado por Daniel Keep e continuado por Lukas Wirth.

## Macros Procedurais para Gerar Código a Partir de Atributos

A segunda forma de macros é a macro procedural, que age mais como uma função (e é um tipo de procedimento). _Macros procedurais_ aceitam algum código como entrada, operam nesse código e produzem algum código como saída, em vez de corresponder a padrões e substituir o código por outro código como fazem as macros declarativas. Os três tipos de macros procedurais são `derive` personalizado, semelhante a atributo e semelhante a função, e todos funcionam de forma semelhante.

Ao criar macros procedurais, as definições devem residir em seu próprio crate com um tipo de crate especial. Isso é por razões técnicas complexas que esperamos eliminar no futuro. Na Listagem 20-36, mostramos como definir uma macro procedural, onde `some_attribute` é um placeholder para usar uma variedade específica de macro.

**Arquivo: src/lib.rs**

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

[Listagem 20-36](#listagem-20-36): Um exemplo de definição de macro procedural

A função que define uma macro procedural recebe um `TokenStream` como entrada e produz um `TokenStream` como saída. O tipo `TokenStream` é definido pelo crate `proc_macro` que vem com Rust e representa uma sequência de tokens. Este é o núcleo da macro: o código-fonte sobre o qual a macro está operando compõe o `TokenStream` de entrada, e o código que a macro produz é o `TokenStream` de saída. A função também tem um atributo anexado a ela que especifica que tipo de macro procedural estamos criando. Podemos ter múltiplos tipos de macros procedurais no mesmo crate.

Vamos olhar os diferentes tipos de macros procedurais. Começaremos com uma macro `derive` personalizada e então explicaremos as pequenas dissimilaridades que tornam as outras formas diferentes.

<a id="how-to-write-a-custom-derive-macro"></a>

## Macros `derive` Personalizadas

Vamos criar um crate chamado `hello_macro` que define uma trait chamada `HelloMacro` com uma função associada chamada `hello_macro`. Em vez de fazer nossos usuários implementarem a trait `HelloMacro` para cada um de seus tipos, forneceremos uma macro procedural para que os usuários possam anotar seu tipo com `#[derive(HelloMacro)]` para obter uma implementação padrão da função `hello_macro`. A implementação padrão imprimirá `Hello, Macro! My name is TypeName!` onde `TypeName` é o nome do tipo no qual esta trait foi definida. Em outras palavras, escreveremos um crate que permite que outro programador escreva código como o da Listagem 20-37 usando nosso crate.

**Arquivo: src/main.rs**

```rust,ignore,does_not_compile
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

[Listagem 20-37](#listagem-20-37): O código que um usuário do nosso crate poderá escrever ao usar nossa macro procedural

Este código imprimirá `Hello, Macro! My name is Pancakes!` quando terminarmos. O primeiro passo é fazer um novo crate de biblioteca, assim:

```bash
$ cargo new hello_macro --lib
```

Em seguida, na Listagem 20-38, definiremos a trait `HelloMacro` e sua função associada.

**Arquivo: src/lib.rs**

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

[Listagem 20-38](#listagem-20-38): Uma trait simples que usaremos com a macro `derive`

Temos uma trait e sua função. Neste ponto, o usuário do nosso crate poderia implementar a trait para alcançar a funcionalidade desejada, como na Listagem 20-39.

**Arquivo: src/main.rs**

```rust,ignore
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}
```

[Listagem 20-39](#listagem-20-39): Como ficaria se os usuários escrevessem uma implementação manual da trait `HelloMacro`

Porém, eles precisariam escrever o bloco de implementação para cada tipo que quisessem usar com `hello_macro`; queremos poupar esse trabalho.

Além disso, ainda não podemos fornecer a função `hello_macro` com implementação padrão que imprimirá o nome do tipo no qual a trait é implementada: Rust não tem capacidades de reflexão, então não pode buscar o nome do tipo em runtime. Precisamos de uma macro para gerar código em compile time.

O próximo passo é definir a macro procedural. No momento em que escrevemos, macros procedurais precisam estar em seu próprio crate. Eventualmente, esta restrição pode ser removida. A convenção para estruturar crates e crates de macro é a seguinte: para um crate chamado `foo`, um crate de macro procedural `derive` personalizado é chamado `foo_derive`. Vamos iniciar um novo crate chamado `hello_macro_derive` dentro do nosso projeto `hello_macro`:

```bash
$ cargo new hello_macro_derive --lib
```

Nossos dois crates estão intimamente relacionados, então criamos o crate de macro procedural dentro do diretório do nosso crate `hello_macro`. Se mudarmos a definição da trait em `hello_macro`, também teremos que mudar a implementação da macro procedural em `hello_macro_derive`. Os dois crates precisarão ser publicados separadamente, e programadores que usam esses crates precisarão adicionar ambos como dependências e trazê-los para o escopo. Poderíamos em vez disso fazer o crate `hello_macro` usar `hello_macro_derive` como dependência e reexportar o código da macro procedural. Porém, a forma como estruturamos o projeto torna possível para programadores usarem `hello_macro` mesmo se não quiserem a funcionalidade `derive`.

Precisamos declarar o crate `hello_macro_derive` como um crate de macro procedural. Também precisaremos de funcionalidade dos crates `syn` e `quote`, como você verá em instantes, então precisamos adicioná-los como dependências. Adicione o seguinte ao arquivo _Cargo.toml_ de `hello_macro_derive`:

**Arquivo: hello_macro_derive/Cargo.toml**

```toml
[lib]
proc-macro = true

[dependencies]
syn = "2.0"
quote = "1.0"
```

Para começar a definir a macro procedural, coloque o código da Listagem 20-40 no arquivo _src/lib.rs_ do crate `hello_macro_derive`. Observe que este código não compilará até adicionarmos uma definição para a função `impl_hello_macro`.

**Arquivo: hello_macro_derive/src/lib.rs**

```rust,ignore,does_not_compile
use proc_macro::TokenStream;
use quote::quote;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate.
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation.
    impl_hello_macro(&ast)
}
```

[Listagem 20-40](#listagem-20-40): Código que a maioria dos crates de macro procedural exigirá para processar código Rust

Observe que dividimos o código na função `hello_macro_derive`, que é responsável por fazer parse do `TokenStream`, e na função `impl_hello_macro`, que é responsável por transformar a árvore de sintaxe: isso torna escrever uma macro procedural mais conveniente. O código na função externa (`hello_macro_derive` neste caso) será o mesmo para quase todo crate de macro procedural que você ver ou criar. O código que você especifica no corpo da função interna (`impl_hello_macro` neste caso) será diferente dependendo do propósito da sua macro procedural.

Introduzimos três crates novos: `proc_macro`, [`syn`][syn] e [`quote`][quote]. O crate `proc_macro` vem com Rust, então não precisamos adicioná-lo às dependências em _Cargo.toml_. O crate `proc_macro` é a API do compilador que nos permite ler e manipular código Rust a partir do nosso código.

O crate `syn` faz parse de código Rust de uma string em uma estrutura de dados sobre a qual podemos realizar operações. O crate `quote` transforma estruturas de dados `syn` de volta em código Rust. Esses crates tornam muito mais simples fazer parse de qualquer tipo de código Rust que possamos querer manipular: escrever um parser completo para código Rust não é tarefa simples.

A função `hello_macro_derive` será chamada quando um usuário da nossa biblioteca especificar `#[derive(HelloMacro)]` em um tipo. Isso é possível porque anotamos a função `hello_macro_derive` aqui com `proc_macro_derive` e especificamos o nome `HelloMacro`, que corresponde ao nome da nossa trait; esta é a convenção que a maioria das macros procedurais segue.

A função `hello_macro_derive` primeiro converte o `input` de um `TokenStream` para uma estrutura de dados que podemos então interpretar e sobre a qual podemos realizar operações. É aqui que `syn` entra em cena. A função `parse` em `syn` recebe um `TokenStream` e retorna uma struct `DeriveInput` representando o código Rust parseado. A Listagem 20-41 mostra as partes relevantes da struct `DeriveInput` que obtemos ao fazer parse da string `struct Pancakes;`.

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

[Listagem 20-41](#listagem-20-41): A instância de `DeriveInput` que obtemos ao fazer parse do código que tem o atributo da macro na Listagem 20-37

Os campos desta struct mostram que o código Rust que parseamos é uma unit struct com o `ident` (_identificador_, ou seja, o nome) de `Pancakes`. Há mais campos nesta struct para descrever todo tipo de código Rust; consulte a [documentação de `syn` para `DeriveInput`][syn-docs] para mais informações.

Em breve definiremos a função `impl_hello_macro`, que é onde construiremos o novo código Rust que queremos incluir. Mas antes de fazer isso, observe que a saída da nossa macro `derive` também é um `TokenStream`. O `TokenStream` retornado é adicionado ao código que os usuários do nosso crate escrevem, então quando eles compilarem seu crate, obterão a funcionalidade extra que fornecemos no `TokenStream` modificado.

Você pode ter notado que estamos chamando `unwrap` para fazer a função `hello_macro_derive` entrar em pânico se a chamada à função `syn::parse` falhar aqui. É necessário que nossa macro procedural entre em pânico em erros porque funções `proc_macro_derive` devem retornar `TokenStream` em vez de `Result` para conformar com a API de macro procedural. Simplificamos este exemplo usando `unwrap`; em código de produção, você deve fornecer mensagens de erro mais específicas sobre o que deu errado usando `panic!` ou `expect`.

Agora que temos o código para transformar o código Rust anotado de um `TokenStream` em uma instância de `DeriveInput`, vamos gerar o código que implementa a trait `HelloMacro` no tipo anotado, como mostrado na Listagem 20-42.

**Arquivo: hello_macro_derive/src/lib.rs**

```rust,ignore
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let generated = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    generated.into()
}
```

[Listagem 20-42](#listagem-20-42): Implementando a trait `HelloMacro` usando o código Rust parseado

Obtemos uma instância da struct `Ident` contendo o nome (identificador) do tipo anotado usando `ast.ident`. A struct na Listagem 20-41 mostra que quando executamos a função `impl_hello_macro` no código da Listagem 20-37, o `ident` que obtemos terá o campo `ident` com um valor de `"Pancakes"`. Assim, a variável `name` na Listagem 20-42 conterá uma instância da struct `Ident` que, quando impressa, será a string `"Pancakes"`, o nome da struct na Listagem 20-37.

A macro `quote!` nos permite definir o código Rust que queremos retornar. O compilador espera algo diferente do resultado direto da execução da macro `quote!`, então precisamos convertê-lo para um `TokenStream`. Fazemos isso chamando o método `into`, que consome esta representação intermediária e retorna um valor do tipo `TokenStream` exigido.

A macro `quote!` também fornece algumas mecânicas de templating muito legais: podemos inserir `#name`, e `quote!` o substituirá pelo valor na variável `name`. Você pode até fazer alguma repetição semelhante à forma como macros regulares funcionam. Consulte [a documentação do crate `quote`][quote-docs] para uma introdução completa.

Queremos que nossa macro procedural gere uma implementação da nossa trait `HelloMacro` para o tipo que o usuário anotou, que podemos obter usando `#name`. A implementação da trait tem a função `hello_macro`, cujo corpo contém a funcionalidade que queremos fornecer: imprimir `Hello, Macro! My name is` e então o nome do tipo anotado.

A macro `stringify!` usada aqui é integrada ao Rust. Ela recebe uma expressão Rust, como `1 + 2`, e em compile time transforma a expressão em um literal de string, como `"1 + 2"`. Isso é diferente de `format!` ou `println!`, que são macros que avaliam a expressão e então transformam o resultado em uma `String`. Há a possibilidade de que a entrada `#name` possa ser uma expressão para imprimir literalmente, então usamos `stringify!`. Usar `stringify!` também economiza uma alocação convertendo `#name` em um literal de string em compile time.

Neste ponto, `cargo build` deve completar com sucesso tanto em `hello_macro` quanto em `hello_macro_derive`. Vamos conectar esses crates ao código da Listagem 20-37 para ver a macro procedural em ação! Crie um novo projeto binário no seu diretório _projects_ usando `cargo new pancakes`. Precisamos adicionar `hello_macro` e `hello_macro_derive` como dependências no _Cargo.toml_ do crate `pancakes`. Se você estiver publicando suas versões de `hello_macro` e `hello_macro_derive` em [crates.io](https://crates.io/), elas seriam dependências regulares; se não, você pode especificá-las como dependências `path` da seguinte forma:

```toml
[dependencies]
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```

Coloque o código da Listagem 20-37 em _src/main.rs_ e execute `cargo run`: deve imprimir `Hello, Macro! My name is Pancakes!`. A implementação da trait `HelloMacro` da macro procedural foi incluída sem o crate `pancakes` precisar implementá-la; o `#[derive(HelloMacro)]` adicionou a implementação da trait.

Em seguida, vamos explorar como os outros tipos de macros procedurais diferem das macros `derive` personalizadas.

## Macros Semelhantes a Atributos

Macros semelhantes a atributos são semelhantes a macros `derive` personalizadas, mas em vez de gerar código para o atributo `derive`, permitem que você crie novos atributos. Elas também são mais flexíveis: `derive` funciona apenas para structs e enums; atributos podem ser aplicados a outros itens também, como funções. Aqui está um exemplo de uso de uma macro semelhante a atributo. Digamos que você tem um atributo chamado `route` que anota funções ao usar um framework de aplicação web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Este atributo `#[route]` seria definido pelo framework como uma macro procedural. A assinatura da função de definição da macro ficaria assim:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Aqui, temos dois parâmetros do tipo `TokenStream`. O primeiro é para o conteúdo do atributo: a parte `GET, "/"`. O segundo é o corpo do item ao qual o atributo está anexado: neste caso, `fn index() {}` e o resto do corpo da função.

Fora isso, macros semelhantes a atributos funcionam da mesma forma que macros `derive` personalizadas: você cria um crate com o tipo de crate `proc-macro` e implementa uma função que gera o código que você quer!

## Macros Semelhantes a Funções

Macros semelhantes a funções definem macros que parecem chamadas de função. De forma semelhante às macros `macro_rules!`, elas são mais flexíveis que funções; por exemplo, podem receber um número desconhecido de argumentos. Porém, macros `macro_rules!` só podem ser definidas usando a sintaxe semelhante a `match` que discutimos na seção [Macros declarativas para metaprogramação geral](#declarative-macros-with-macro_rules-for-general-metaprogramming) anteriormente. Macros semelhantes a funções recebem um parâmetro `TokenStream`, e sua definição manipula esse `TokenStream` usando código Rust como os outros dois tipos de macros procedurais fazem. Um exemplo de macro semelhante a função é uma macro `sql!` que poderia ser chamada assim:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Esta macro faria parse da instrução SQL dentro dela e verificaria se está sintaticamente correta, o que é processamento muito mais complexo do que uma macro `macro_rules!` pode fazer. A macro `sql!` seria definida assim:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Esta definição é semelhante à assinatura da macro `derive` personalizada: recebemos os tokens que estão dentro dos parênteses e retornamos o código que queríamos gerar.

## Resumo

Ufa! Agora você tem alguns recursos do Rust na sua caixa de ferramentas que provavelmente não usará com frequência, mas saberá que estão disponíveis em circunstâncias muito particulares. Introduzimos vários tópicos complexos para que, quando você os encontrar em sugestões de mensagens de erro ou em outros códigos, reconheça esses conceitos e essa sintaxe. Use este capítulo como referência para guiá-lo a soluções.

Em seguida, colocaremos tudo o que discutimos ao longo do livro em prática e faremos mais um projeto!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
