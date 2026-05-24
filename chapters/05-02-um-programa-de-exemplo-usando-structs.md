---
title: "Um programa de exemplo usando structs"
chapter_code: 05-02
slug: um-programa-de-exemplo-usando-structs
---

# Um Programa de Exemplo Usando Structs

Para entender quando podemos querer usar structs, vamos escrever um programa que calcula a área de um retângulo. Começaremos usando variáveis simples e depois refatoraremos o programa até usarmos structs.

Vamos criar um novo projeto binário com Cargo chamado _rectangles_ que receberá a largura e a altura de um retângulo especificadas em pixels e calculará a área do retângulo. A Listagem 5-8 mostra um programa curto com uma forma de fazer exatamente isso no arquivo `src/main.rs` do projeto.

**Arquivo: src/main.rs**

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

[Listagem 5-8](#listagem-5-8): Calculando a área de um retângulo especificado por variáveis separadas de largura e altura

Agora, execute este programa com `cargo run`:

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/rectangles`
The area of the rectangle is 1500 square pixels.
```

Este código consegue calcular a área do retângulo chamando a função `area` com cada dimensão, mas podemos fazer mais para deixar esse código claro e legível.

O problema com este código fica evidente na assinatura de `area`:

```rust
fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

A função `area` deveria calcular a área de um retângulo, mas a função que escrevemos tem dois parâmetros, e em nenhum lugar do programa fica claro que os parâmetros estão relacionados. Seria mais legível e mais fácil de manter agrupar largura e altura. Já discutimos uma forma de fazer isso na seção O tipo tupla do Capítulo 3: usando tuplas.

### Refatorando com tuplas

A Listagem 5-9 mostra outra versão do nosso programa que usa tuplas.

**Arquivo: src/main.rs**

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

[Listagem 5-9](#listagem-5-9): Especificando a largura e a altura do retângulo com uma tupla

De certa forma, este programa é melhor. As tuplas nos permitem adicionar um pouco de estrutura, e agora passamos apenas um argumento. Mas, de outra forma, esta versão é menos clara: tuplas não nomeiam seus elementos, então precisamos indexar as partes da tupla, o que torna nosso cálculo menos óbvio.

Confundir largura e altura não afetaria o cálculo da área, mas se quiséssemos desenhar o retângulo na tela, isso importaria! Teríamos que lembrar que `width` é o índice `0` da tupla e `height` é o índice `1`. Isso seria ainda mais difícil para outra pessoa descobrir e manter em mente se fosse usar nosso código. Como não transmitimos o significado dos nossos dados no código, agora é mais fácil introduzir erros.

### Refatorando com structs

Usamos structs para adicionar significado rotulando os dados. Podemos transformar a tupla que estamos usando em uma struct com um nome para o todo e nomes para as partes, como mostrado na Listagem 5-10.

**Arquivo: src/main.rs**

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

[Listagem 5-10](#listagem-5-10): Definindo uma struct `Rectangle`

Aqui, definimos uma struct e a nomeamos `Rectangle`. Dentro das chaves, definimos os campos `width` e `height`, ambos do tipo `u32`. Em seguida, em `main`, criamos uma instância particular de `Rectangle` com largura `30` e altura `50`.

Nossa função `area` agora é definida com um parâmetro, que nomeamos `rectangle`, cujo tipo é um borrow imutável de uma instância da struct `Rectangle`. Como mencionado no Capítulo 4, queremos fazer borrow da struct em vez de tomar ownership dela. Dessa forma, `main` mantém seu ownership e pode continuar usando `rect1`, que é o motivo de usarmos `&` na assinatura da função e onde chamamos a função.

A função `area` acessa os campos `width` e `height` da instância de `Rectangle` (observe que acessar campos de uma instância de struct emprestada não move os valores dos campos, por isso você frequentemente vê borrows de structs). A assinatura de `area` agora diz exatamente o que queremos dizer: calcular a área de um `Rectangle`, usando seus campos `width` e `height`. Isso transmite que largura e altura estão relacionadas entre si e dá nomes descritivos aos valores em vez de usar os índices `0` e `1` da tupla. Isso é uma vitória para a clareza.

### Adicionando funcionalidade com traits derivadas

Seria útil poder imprimir uma instância de `Rectangle` enquanto depuramos nosso programa e ver os valores de todos os seus campos. A Listagem 5-11 tenta usar a macro `println!` como fizemos em capítulos anteriores. Isso não funcionará, porém.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1}");
}
```

[Listagem 5-11](#listagem-5-11): Tentando imprimir uma instância de `Rectangle`

Quando compilamos este código, obtemos um erro com esta mensagem central:

```text
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```

A macro `println!` pode fazer muitos tipos de formatação e, por padrão, as chaves dizem a `println!` para usar a formatação conhecida como `Display`: saída destinada ao consumo direto pelo usuário final. Os tipos primitivos que vimos até agora implementam `Display` por padrão porque há apenas uma forma de mostrar um `1` ou qualquer outro tipo primitivo a um usuário. Mas com structs, a forma como `println!` deveria formatar a saída é menos clara porque há mais possibilidades de exibição: você quer vírgulas ou não? Quer imprimir as chaves? Todos os campos devem ser mostrados? Devido a essa ambiguidade, o Rust não tenta adivinhar o que queremos, e structs não têm uma implementação fornecida de `Display` para usar com `println!` e o placeholder `{}`.

Se continuarmos lendo os erros, encontraremos esta nota útil:

```text
= note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```

Vamos tentar! A chamada da macro `println!` agora ficará assim: `println!("rect1 is {rect1:?}");`. Colocar o especificador `:?` dentro das chaves diz a `println!` que queremos usar um formato de saída chamado `Debug`. A trait `Debug` nos permite imprimir nossa struct de uma forma útil para desenvolvedores, para que possamos ver seu valor enquanto depuramos nosso código.

Compile o código com essa alteração. Droga! Ainda obtemos um erro:

```text
error[E0277]: `Rectangle` doesn't implement `Debug`
```

Mas, novamente, o compilador nos dá uma nota útil:

```text
= note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
```

O Rust _inclui_ funcionalidade para imprimir informações de depuração, mas precisamos optar explicitamente por tornar essa funcionalidade disponível para nossa struct. Para isso, adicionamos o atributo externo `#[derive(Debug)]` logo antes da definição da struct, como mostrado na Listagem 5-12.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1:?}");
}
```

[Listagem 5-12](#listagem-5-12): Adicionando o atributo para derivar a trait `Debug` e imprimindo a instância de `Rectangle` usando formatação de depuração

Agora, quando executamos o programa, não obtemos erros e vemos a seguinte saída:

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle { width: 30, height: 50 }
```

Ótimo! Não é a saída mais bonita, mas mostra os valores de todos os campos desta instância, o que definitivamente ajudaria durante a depuração. Quando temos structs maiores, é útil ter uma saída um pouco mais fácil de ler; nesses casos, podemos usar `{:#?}` em vez de `{:?}` na string do `println!`. Neste exemplo, usar o estilo `{:#?}` produzirá a seguinte saída:

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

Outra forma de imprimir um valor usando o formato `Debug` é usar a macro `dbg!`, que toma ownership de uma expressão (em oposição a `println!`, que recebe uma referência), imprime o arquivo e o número da linha onde a chamada da macro `dbg!` ocorre no seu código junto com o valor resultante dessa expressão e devolve o ownership do valor.

> **Nota:** Chamar a macro `dbg!` imprime no fluxo de console de erro padrão (`stderr`), em oposição a `println!`, que imprime no fluxo de saída padrão (`stdout`). Falaremos mais sobre `stderr` e `stdout` na seção Redirecionando erros para a saída de erro padrão do Capítulo 12.

Aqui está um exemplo em que nos interessa o valor atribuído ao campo `width`, bem como o valor da struct inteira em `rect1`:

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

Podemos colocar `dbg!` em torno da expressão `30 * scale` e, como `dbg!` devolve o ownership do valor da expressão, o campo `width` receberá o mesmo valor que se não tivéssemos a chamada a `dbg!` ali. Não queremos que `dbg!` tome ownership de `rect1`, então usamos uma referência a `rect1` na próxima chamada. Veja como fica a saída deste exemplo:

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10:16] 30 * scale = 60
[src/main.rs:14:5] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

Podemos ver que a primeira parte da saída veio da linha 10 de `src/main.rs`, onde estamos depurando a expressão `30 * scale`, e seu valor resultante é `60` (a formatação `Debug` implementada para inteiros é imprimir apenas seu valor). A chamada a `dbg!` na linha 14 de `src/main.rs` imprime o valor de `&rect1`, que é a struct `Rectangle`. Essa saída usa a formatação `Debug` mais legível do tipo `Rectangle`. A macro `dbg!` pode ser muito útil quando você está tentando descobrir o que seu código está fazendo!

Além da trait `Debug`, o Rust fornece várias traits para usarmos com o atributo `derive` que podem adicionar comportamento útil aos nossos tipos personalizados. Essas traits e seus comportamentos estão listados no [Apêndice C](/livro/cap22-03-traits-derivaveis). Cobriremos como implementar essas traits com comportamento personalizado, bem como como criar suas próprias traits, no Capítulo 10. Também existem muitos outros atributos além de `derive`; para mais informações, consulte a seção Attributes da Referência do Rust.

Nossa função `area` é muito específica: ela só calcula a área de retângulos. Seria útil vincular esse comportamento mais de perto à nossa struct `Rectangle`, porque ele não funcionará com nenhum outro tipo. Vamos ver como podemos continuar refatorando este código transformando a função `area` em um método `area` definido no nosso tipo `Rectangle`.
