---
title: "Tipos de dados genéricos"
chapter_code: 10-01
slug: tipos-de-dados-genericos
---

# Tipos de Dados Genéricos

Usamos generics para criar definições de itens como assinaturas de função ou structs, que podemos então usar com muitos tipos de dados concretos diferentes. Primeiro veremos como definir funções, structs, enums e métodos usando generics. Depois, discutiremos como generics afetam o desempenho do código.

### Em definições de função

Ao definir uma função que usa generics, colocamos os generics na assinatura da função onde normalmente especificaríamos os tipos de dados dos parâmetros e do valor de retorno. Isso torna nosso código mais flexível e oferece mais funcionalidade a quem chama nossa função, evitando duplicação de código.

Continuando com nossa função `largest`, a Listagem 10-4 mostra duas funções que encontram o maior valor em um slice. Em seguida, as combinaremos em uma única função que usa generics.

**Arquivo: src/main.rs**

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {result}");

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {result}");
}
```

<a id="listagem-10-4"></a>

[Listagem 10-4](#listagem-10-4): Duas funções que diferem apenas nos nomes e nos tipos em suas assinaturas

A função `largest_i32` é a que extraímos na Listagem 10-3 e que encontra o maior `i32` em um slice. A função `largest_char` encontra o maior `char` em um slice. Os corpos das funções têm o mesmo código; vamos eliminar a duplicação introduzindo um parâmetro de tipo genérico em uma única função.

Para parametrizar os tipos em uma nova função única, precisamos nomear o parâmetro de tipo, assim como fazemos com parâmetros de valor em uma função. Você pode usar qualquer identificador como nome de parâmetro de tipo, mas usaremos `T` porque, por convenção, nomes de parâmetros de tipo em Rust são curtos, muitas vezes uma única letra, e a convenção de nomenclatura de tipos em Rust é UpperCamelCase. Abreviação de _type_, `T` é a escolha padrão da maioria dos programadores Rust.

Quando usamos um parâmetro no corpo da função, temos de declarar o nome do parâmetro na assinatura para que o compilador saiba o que esse nome significa. Da mesma forma, quando usamos um nome de parâmetro de tipo em uma assinatura de função, temos de declarar o nome do parâmetro de tipo antes de usá-lo. Para definir a função genérica `largest`, colocamos declarações de nome de tipo dentro de colchetes angulares, `<>`, entre o nome da função e a lista de parâmetros, assim:

```rust
fn largest<T>(list: &[T]) -> &T {
```

Lemos esta definição como “A função `largest` é genérica sobre algum tipo `T`.” Esta função tem um parâmetro chamado `list`, que é um slice de valores do tipo `T`. A função `largest` retornará uma referência a um valor do mesmo tipo `T`.

A Listagem 10-5 mostra a definição combinada da função `largest` usando o tipo de dado genérico em sua assinatura. A listagem também mostra como podemos chamar a função com um slice de valores `i32` ou `char`. Observe que este código ainda não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {result}");

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {result}");
}
```

<a id="listagem-10-5"></a>

[Listagem 10-5](#listagem-10-5): A função `largest` usando parâmetros de tipo genérico; isso ainda não compila

Se compilarmos este código agora, obteremos este erro:

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T` with trait `PartialOrd`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

O texto de ajuda menciona `std::cmp::PartialOrd`, que é uma trait, e falaremos sobre traits na próxima seção. Por enquanto, saiba que este erro indica que o corpo de `largest` não funcionará para todos os tipos possíveis que `T` poderia ser. Como queremos comparar valores do tipo `T` no corpo, só podemos usar tipos cujos valores possam ser ordenados. Para habilitar comparações, a biblioteca padrão tem a trait `std::cmp::PartialOrd`, que você pode implementar em tipos (veja o [Apêndice C](/livro/cap22-03-traits-derivaveis) para mais sobre esta trait). Para corrigir a Listagem 10-5, podemos seguir a sugestão do texto de ajuda e restringir os tipos válidos para `T` apenas àqueles que implementam `PartialOrd`. A listagem então compilará, porque a biblioteca padrão implementa `PartialOrd` tanto em `i32` quanto em `char`.

### Em definições de struct

Também podemos definir structs para usar um parâmetro de tipo genérico em um ou mais campos usando a sintaxe `<>`. A Listagem 10-6 define uma struct `Point<T>` para armazenar valores de coordenadas `x` e `y` de qualquer tipo.

**Arquivo: src/main.rs**

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<a id="listagem-10-6"></a>

[Listagem 10-6](#listagem-10-6): Uma struct `Point<T>` que armazena valores `x` e `y` do tipo `T`

A sintaxe para usar generics em definições de struct é semelhante à usada em definições de função. Primeiro, declaramos o nome do parâmetro de tipo dentro de colchetes angulares logo após o nome da struct. Depois, usamos o tipo genérico na definição da struct onde, caso contrário, especificaríamos tipos de dados concretos.

Observe que, como usamos apenas um tipo genérico para definir `Point<T>`, esta definição diz que a struct `Point<T>` é genérica sobre algum tipo `T`, e os campos `x` e `y` são _ambos_ desse mesmo tipo, seja qual for. Se criarmos uma instância de `Point<T>` que tenha valores de tipos diferentes, como na Listagem 10-7, nosso código não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<a id="listagem-10-7"></a>

[Listagem 10-7](#listagem-10-7): Os campos `x` e `y` devem ser do mesmo tipo porque ambos têm o mesmo tipo de dado genérico `T`

Neste exemplo, quando atribuímos o valor inteiro `5` a `x`, informamos ao compilador que o tipo genérico `T` será um inteiro para esta instância de `Point<T>`. Depois, quando especificamos `4.0` para `y`, que definimos para ter o mesmo tipo que `x`, obteremos um erro de incompatibilidade de tipos como este:

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

Para definir uma struct `Point` em que `x` e `y` sejam ambos genéricos, mas possam ter tipos diferentes, podemos usar vários parâmetros de tipo genérico. Por exemplo, na Listagem 10-8, mudamos a definição de `Point` para ser genérica sobre os tipos `T` e `U`, onde `x` é do tipo `T` e `y` é do tipo `U`.

**Arquivo: src/main.rs**

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<a id="listagem-10-8"></a>

[Listagem 10-8](#listagem-10-8): Um `Point<T, U>` genérico sobre dois tipos para que `x` e `y` possam ser valores de tipos diferentes

Agora todas as instâncias de `Point` mostradas são permitidas! Você pode usar quantos parâmetros de tipo genérico quiser em uma definição, mas usar mais do que alguns torna seu código difícil de ler. Se perceber que precisa de muitos tipos genéricos no código, pode indicar que ele precisa ser reestruturado em partes menores.

### Em definições de enum

Como fizemos com structs, podemos definir enums para armazenar tipos de dados genéricos em suas variantes. Vejamos novamente o enum `Option<T>` que a biblioteca padrão fornece, que usamos no Capítulo 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Esta definição agora deve fazer mais sentido. Como você pode ver, o enum `Option<T>` é genérico sobre o tipo `T` e tem duas variantes: `Some`, que contém um valor do tipo `T`, e uma variante `None` que não contém nenhum valor. Ao usar o enum `Option<T>`, podemos expressar o conceito abstrato de um valor opcional e, como `Option<T>` é genérico, podemos usar essa abstração não importa qual seja o tipo do valor opcional.

Enums também podem usar vários tipos genéricos. A definição do enum `Result` que usamos no Capítulo 9 é um exemplo:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

O enum `Result` é genérico sobre dois tipos, `T` e `E`, e tem duas variantes: `Ok`, que contém um valor do tipo `T`, e `Err`, que contém um valor do tipo `E`. Esta definição torna conveniente usar o enum `Result` em qualquer lugar em que tenhamos uma operação que pode ter sucesso (retornar um valor de algum tipo `T`) ou falhar (retornar um erro de algum tipo `E`). Na verdade, foi o que usamos para abrir um arquivo na Listagem 9-3, onde `T` foi preenchido com o tipo `std::fs::File` quando o arquivo foi aberto com sucesso e `E` foi preenchido com o tipo `std::io::Error` quando houve problemas ao abrir o arquivo.

Quando reconhecer situações no código com várias definições de struct ou enum que diferem apenas nos tipos dos valores que armazenam, você pode evitar duplicação usando tipos genéricos.

### Em definições de método

Podemos implementar métodos em structs e enums (como fizemos no Capítulo 5) e usar tipos genéricos em suas definições também. A Listagem 10-9 mostra a struct `Point<T>` que definimos na Listagem 10-6 com um método chamado `x` implementado nela.

**Arquivo: src/main.rs**

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<a id="listagem-10-9"></a>

[Listagem 10-9](#listagem-10-9): Implementando um método chamado `x` na struct `Point<T>` que retornará uma referência ao campo `x` do tipo `T`

Aqui, definimos um método chamado `x` em `Point<T>` que retorna uma referência aos dados no campo `x`.

Observe que temos de declarar `T` logo após `impl` para podermos usar `T` e especificar que estamos implementando métodos no tipo `Point<T>`. Ao declarar `T` como tipo genérico após `impl`, o Rust identifica que o tipo entre colchetes angulares em `Point` é um tipo genérico e não um tipo concreto. Poderíamos ter escolhido um nome diferente para este parâmetro genérico do que o declarado na definição da struct, mas usar o mesmo nome é convencional. Se você escrever um método dentro de um `impl` que declara um tipo genérico, esse método será definido em qualquer instância do tipo, não importa qual tipo concreto substitua o tipo genérico.

Também podemos especificar restrições em tipos genéricos ao definir métodos no tipo. Por exemplo, poderíamos implementar métodos apenas em instâncias de `Point<f32>` em vez de em instâncias de `Point<T>` com qualquer tipo genérico. Na Listagem 10-10, usamos o tipo concreto `f32`, o que significa que não declaramos nenhum tipo após `impl`.

**Arquivo: src/main.rs**

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<a id="listagem-10-10"></a>

[Listagem 10-10](#listagem-10-10): Um bloco `impl` que se aplica apenas a uma struct com um tipo concreto particular para o parâmetro de tipo genérico `T`

Este código significa que o tipo `Point<f32>` terá um método `distance_from_origin`; outras instâncias de `Point<T>` em que `T` não seja do tipo `f32` não terão este método definido. O método mede a que distância nosso ponto está do ponto nas coordenadas (0.0, 0.0) e usa operações matemáticas disponíveis apenas para tipos de ponto flutuante.

Parâmetros de tipo genérico em uma definição de struct nem sempre são os mesmos que você usa nas assinaturas de método dessa mesma struct. A Listagem 10-11 usa os tipos genéricos `X1` e `Y1` para a struct `Point` e `X2` e `Y2` para a assinatura do método `mixup`, para deixar o exemplo mais claro. O método cria uma nova instância de `Point` com o valor `x` do `Point` `self` (do tipo `X1`) e o valor `y` do `Point` passado (do tipo `Y2`).

**Arquivo: src/main.rs**

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<a id="listagem-10-11"></a>

[Listagem 10-11](#listagem-10-11): Um método que usa tipos genéricos diferentes da definição de sua struct

Em `main`, definimos um `Point` que tem um `i32` para `x` (com valor `5`) e um `f64` para `y` (com valor `10.4`). A variável `p2` é uma struct `Point` que tem um slice de string para `x` (com valor `"Hello"`) e um `char` para `y` (com valor `c`). Chamar `mixup` em `p1` com o argumento `p2` nos dá `p3`, que terá um `i32` para `x` porque `x` veio de `p1`. A variável `p3` terá um `char` para `y` porque `y` veio de `p2`. A chamada à macro `println!` imprimirá `p3.x = 5, p3.y = c`.

O propósito deste exemplo é demonstrar uma situação em que alguns parâmetros genéricos são declarados com `impl` e outros na definição do método. Aqui, os parâmetros genéricos `X1` e `Y1` são declarados após `impl` porque acompanham a definição da struct. Os parâmetros genéricos `X2` e `Y2` são declarados após `fn mixup` porque são relevantes apenas para o método.

### Desempenho do código que usa generics

Você pode estar se perguntando se há custo em tempo de execução ao usar parâmetros de tipo genérico. A boa notícia é que usar tipos genéricos não tornará seu programa mais lento do que seria com tipos concretos.

O Rust consegue isso realizando monomorfização do código que usa generics em tempo de compilação. _Monomorfização_ é o processo de transformar código genérico em código específico preenchendo os tipos concretos usados quando compilado. Nesse processo, o compilador faz o oposto dos passos que usamos para criar a função genérica na Listagem 10-5: o compilador examina todos os lugares onde código genérico é chamado e gera código para os tipos concretos com os quais o código genérico é chamado.

Vejamos como isso funciona usando o enum genérico `Option<T>` da biblioteca padrão:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Quando o Rust compila este código, ele realiza monomorfização. Durante esse processo, o compilador lê os valores usados em instâncias de `Option<T>` e identifica dois tipos de `Option<T>`: um é `i32` e o outro é `f64`. Assim, expande a definição genérica de `Option<T>` em duas definições especializadas para `i32` e `f64`, substituindo a definição genérica pelas específicas.

A versão monomorfizada do código parece semelhante ao seguinte (o compilador usa nomes diferentes dos que usamos aqui para ilustração):

**Arquivo: src/main.rs**

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

O `Option<T>` genérico é substituído pelas definições específicas criadas pelo compilador. Como o Rust compila código genérico em código que especifica o tipo em cada instância, não pagamos custo em tempo de execução por usar generics. Quando o código executa, ele se comporta como se tivéssemos duplicado cada definição manualmente. O processo de monomorfização torna os generics do Rust extremamente eficientes em tempo de execução.
