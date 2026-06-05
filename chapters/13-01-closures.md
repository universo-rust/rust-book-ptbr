---
title: "Closures"
chapter_code: 13-01
slug: closures
---

# Closures

As closures do Rust são funções anônimas que você pode salvar em uma variável ou passar como argumentos para outras funções. Você pode criar a closure em um lugar e depois chamá-la em outro para avaliá-la em um contexto diferente. Diferente de funções, closures podem capturar valores do escopo em que são definidas. Demonstraremos como esses recursos de closures permitem reutilização de código e personalização de comportamento.

## Capturando o Ambiente

Primeiro examinaremos como podemos usar closures para capturar valores do ambiente em que são definidas para uso posterior. Aqui está o cenário: de vez em quando, nossa empresa de camisetas dá uma camiseta exclusiva de edição limitada para alguém da nossa lista de e-mails como promoção. As pessoas na lista podem opcionalmente adicionar sua cor favorita ao perfil. Se a pessoa escolhida para uma camiseta grátis tiver a cor favorita definida, ela recebe essa cor. Se a pessoa não especificou uma cor favorita, recebe a cor que a empresa tem em maior quantidade no momento.

Há muitas formas de implementar isso. Para este exemplo, usaremos um enum chamado `ShirtColor` com as variantes `Red` e `Blue` (limitando o número de cores disponíveis por simplicidade). Representamos o estoque da empresa com uma struct `Inventory` que tem um campo chamado `shirts` contendo um `Vec<ShirtColor>` representando as cores de camisetas atualmente em estoque. O método `giveaway` definido em `Inventory` obtém a preferência opcional de cor da camiseta do ganhador da camiseta grátis e retorna a cor que a pessoa receberá. Esta configuração é mostrada na Listagem 13-1.

**Arquivo: src/main.rs**

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```

<a id="listagem-13-1"></a>

[Listagem 13-1](#listagem-13-1): Situação de brinde da empresa de camisetas

O `store` definido em `main` tem duas camisetas azuis e uma vermelha restantes para distribuir nesta promoção de edição limitada. Chamamos o método `giveaway` para um usuário com preferência por camiseta vermelha e para um usuário sem preferência.

Novamente, este código poderia ser implementado de muitas formas e, aqui, para focar em closures, nos limitamos a conceitos que você já aprendeu, exceto o corpo do método `giveaway`, que usa uma closure. No método `giveaway`, obtemos a preferência do usuário como parâmetro do tipo `Option<ShirtColor>` e chamamos o método `unwrap_or_else` em `user_preference`. O método [`unwrap_or_else` em `Option<T>`][unwrap-or-else] é definido pela biblioteca padrão. Ele recebe um argumento: uma closure sem argumentos que retorna um valor `T` (o mesmo tipo armazenado na variante `Some` de `Option<T>`, neste caso `ShirtColor`). Se `Option<T>` for a variante `Some`, `unwrap_or_else` retorna o valor de dentro do `Some`. Se `Option<T>` for a variante `None`, `unwrap_or_else` chama a closure e retorna o valor retornado pela closure.

Especificamos a expressão de closure `|| self.most_stocked()` como argumento de `unwrap_or_else`. Esta é uma closure que não recebe parâmetros (se a closure tivesse parâmetros, eles apareceriam entre as duas barras verticais). O corpo da closure chama `self.most_stocked()`. Estamos definindo a closure aqui, e a implementação de `unwrap_or_else` avaliará a closure depois, se o resultado for necessário.

Executar este código imprime o seguinte:

```console
$ cargo run
   Compiling shirt-company v0.1.0 (file:///projects/shirt-company)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/shirt-company`
The user with preference Some(Red) gets Red
The user with preference None gets Blue
```

Um aspecto interessante aqui é que passamos uma closure que chama `self.most_stocked()` na instância atual de `Inventory`. A biblioteca padrão não precisou saber nada sobre os tipos `Inventory` ou `ShirtColor` que definimos, nem sobre a lógica que queremos usar neste cenário. A closure captura uma referência imutável à instância `self` de `Inventory` e a passa junto com o código que especificamos para o método `unwrap_or_else`. Funções, por outro lado, não conseguem capturar seu ambiente dessa forma.

## Inferindo e Anotando Tipos de Closure

Há mais diferenças entre funções e closures. Closures geralmente não exigem que você anote os tipos dos parâmetros ou do valor de retorno como funções `fn` fazem. Anotações de tipo são exigidas em funções porque os tipos fazem parte de uma interface explícita exposta aos seus usuários. Definir esta interface rigidamente é importante para garantir que todos concordem sobre quais tipos de valores uma função usa e retorna. Closures, por outro lado, não são usadas em uma interface exposta assim: são armazenadas em variáveis e usadas sem nomeá-las nem expô-las aos usuários da nossa biblioteca.

Closures costumam ser curtas e relevantes apenas em um contexto estreito, em vez de em qualquer cenário arbitrário. Dentro desses contextos limitados, o compilador pode inferir os tipos dos parâmetros e do tipo de retorno, de forma semelhante a como consegue inferir os tipos da maioria das variáveis (há casos raros em que o compilador também precisa de anotações de tipo em closures).

Como com variáveis, podemos adicionar anotações de tipo se quisermos aumentar a explicitude e a clareza ao custo de ser mais verboso do que o estritamente necessário. Anotar os tipos de uma closure ficaria como a definição mostrada na Listagem 13-2. Neste exemplo, estamos definindo uma closure e armazenando-a em uma variável, em vez de definir a closure no ponto em que a passamos como argumento, como fizemos na Listagem 13-1.

**Arquivo: src/main.rs**

```rust
use std::thread;
use std::time::Duration;

fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_closure(intensity));
        println!("Next, do {} situps!", expensive_closure(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
}

fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(simulated_user_specified_value, simulated_random_number);
}
```

<a id="listagem-13-2"></a>

[Listagem 13-2](#listagem-13-2): Adicionando anotações de tipo opcionais do parâmetro e do valor de retorno na closure

Com anotações de tipo adicionadas, a sintaxe de closures se parece mais com a sintaxe de funções. Aqui, definimos uma função que adiciona 1 ao seu parâmetro e uma closure com o mesmo comportamento, para comparação. Adicionamos alguns espaços para alinhar as partes relevantes. Isso ilustra como a sintaxe de closure é semelhante à sintaxe de função, exceto pelo uso de barras verticais e pela quantidade de sintaxe que é opcional:

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

A primeira linha mostra uma definição de função e a segunda mostra uma definição de closure totalmente anotada. Na terceira linha, removemos as anotações de tipo da definição de closure. Na quarta linha, removemos as chaves, que são opcionais porque o corpo da closure tem apenas uma expressão. Todas são definições válidas que produzirão o mesmo comportamento quando chamadas. As linhas `add_one_v3` e `add_one_v4` exigem que as closures sejam avaliadas para compilar, porque os tipos serão inferidos pelo uso. Isso é semelhante a `let v = Vec::new();` precisar de anotações de tipo ou de valores de algum tipo inseridos no `Vec` para o Rust inferir o tipo.

Para definições de closure, o compilador inferirá um tipo concreto para cada parâmetro e para o valor de retorno. Por exemplo, a Listagem 13-3 mostra a definição de uma closure curta que apenas retorna o valor que recebe como parâmetro. Esta closure não é muito útil, exceto para os propósitos deste exemplo. Note que não adicionamos anotações de tipo à definição. Como não há anotações de tipo, podemos chamar a closure com qualquer tipo, o que fizemos aqui com `String` na primeira vez. Se então tentarmos chamar `example_closure` com um inteiro, obteremos um erro.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
}
```

<a id="listagem-13-3"></a>

[Listagem 13-3](#listagem-13-3): Tentativa de chamar uma closure cujos tipos foram inferidos com dois tipos diferentes

O compilador nos dá este erro:

```console
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |             --------------- ^ expected `String`, found integer
  |             |
  |             arguments to this function are incorrect
  |
note: expected because the closure was earlier called with an argument of type `String`
 --> src/main.rs:4:29
  |
4 |     let s = example_closure(String::from("hello"));
  |             --------------- ^^^^^^^^^^^^^^^^^^^^^ expected because this argument is of type `String`
  |             |
  |             in this closure call
note: closure parameter defined here
 --> src/main.rs:2:28
  |
2 |     let example_closure = |x| x;
  |                            ^
help: try using a conversion method
  |
5 |     let n = example_closure(5.to_string());
  |                              ++++++++++++

For more information about this error, try `rustc --explain E0308`.
error: could not compile `closure-example` (bin "closure-example") due to 1 previous error
```

Na primeira vez que chamamos `example_closure` com o valor `String`, o compilador infere o tipo de `x` e o tipo de retorno da closure como `String`. Esses tipos ficam então fixados na closure em `example_closure`, e obtemos um erro de tipo quando tentamos usar um tipo diferente com a mesma closure.

## Capturando referências ou movendo posse

Closures podem capturar valores de seu ambiente de três formas, que correspondem diretamente às três formas como uma função pode receber um parâmetro: emprestar imutavelmente, emprestar mutavelmente e tomar posse. A closure decidirá qual usar com base no que o corpo da função faz com os valores capturados.

Na Listagem 13-4, definimos uma closure que captura uma referência imutável ao vetor chamado `list` porque só precisa de uma referência imutável para imprimir o valor.

**Arquivo: src/main.rs**

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    let only_borrows = || println!("From closure: {list:?}");

    println!("Before calling closure: {list:?}");
    only_borrows();
    println!("After calling closure: {list:?}");
}
```

<a id="listagem-13-4"></a>

[Listagem 13-4](#listagem-13-4): Definindo e chamando uma closure que captura uma referência imutável

Este exemplo também ilustra que uma variável pode se ligar a uma definição de closure, e podemos depois chamar a closure usando o nome da variável e parênteses como se o nome da variável fosse um nome de função.

Como podemos ter várias referências imutáveis a `list` ao mesmo tempo, `list` ainda é acessível do código antes da definição da closure, depois da definição da closure mas antes da closure ser chamada, e depois da closure ser chamada. Este código compila, executa e imprime:

```console
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
```

Em seguida, na Listagem 13-5, mudamos o corpo da closure para adicionar um elemento ao vetor `list`. A closure agora captura uma referência mutável.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {list:?}");
}
```

<a id="listagem-13-5"></a>

[Listagem 13-5](#listagem-13-5): Definindo e chamando uma closure que captura uma referência mutável

Este código compila, executa e imprime:

```console
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
```

Note que não há mais um `println!` entre a definição e a chamada da closure `borrows_mutably`: quando `borrows_mutably` é definida, ela captura uma referência mutável a `list`. Não usamos a closure novamente depois que ela é chamada, então o empréstimo mutável termina. Entre a definição da closure e a chamada da closure, um empréstimo imutável para imprimir não é permitido, porque nenhum outro empréstimo é permitido quando há um empréstimo mutável. Tente adicionar um `println!` ali para ver qual mensagem de erro você obtém!

Se quiser forçar a closure a tomar posse dos valores que usa no ambiente mesmo que o corpo da closure não precise estritamente de posse, você pode usar a palavra-chave `move` antes da lista de parâmetros.

Esta técnica é mais útil ao passar uma closure para uma nova thread para mover os dados de forma que sejam possuídos pela nova thread. Discutiremos threads e por que você as usaria em detalhes no Capítulo 16, quando falarmos de concorrência, mas por agora vamos explorar brevemente criar uma nova thread usando uma closure que precisa da palavra-chave `move`. A Listagem 13-6 mostra a Listagem 13-4 modificada para imprimir o vetor em uma nova thread em vez da thread principal.

**Arquivo: src/main.rs**

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    thread::spawn(move || println!("From thread: {list:?}"))
        .join()
        .unwrap();
}
```

<a id="listagem-13-6"></a>

[Listagem 13-6](#listagem-13-6): Usando `move` para forçar a closure da thread a tomar posse de `list`

Criamos uma nova thread, passando à thread uma closure para executar como argumento. O corpo da closure imprime a lista. Na Listagem 13-4, a closure só capturava `list` usando uma referência imutável porque esse é o menor acesso a `list` necessário para imprimi-la. Neste exemplo, embora o corpo da closure ainda precise apenas de uma referência imutável, precisamos especificar que `list` deve ser movida para a closure colocando a palavra-chave `move` no início da definição da closure. Se a thread principal executasse mais operações antes de chamar `join` na nova thread, a nova thread poderia terminar antes do restante da thread principal, ou a thread principal poderia terminar primeiro. Se a thread principal mantivesse posse de `list` mas terminasse antes da nova thread e descartasse `list`, a referência imutável na thread seria inválida. Portanto, o compilador exige que `list` seja movida para a closure passada à nova thread para que a referência seja válida. Tente remover a palavra-chave `move` ou usar `list` na thread principal depois que a closure for definida para ver quais erros do compilador você obtém!

## Movendo valores capturados para fora de closures

Depois que uma closure capturou uma referência ou tomou posse de um valor do ambiente onde a closure é definida (afetando assim o que, se algo, é movido _para dentro_ da closure), o código no corpo da closure define o que acontece com as referências ou valores quando a closure é avaliada depois (afetando assim o que, se algo, é movido _para fora_ da closure).

Um corpo de closure pode fazer qualquer um dos seguintes: mover um valor capturado para fora da closure, mutar o valor capturado, nem mover nem mutar o valor, ou não capturar nada do ambiente para começar.

A forma como uma closure captura e trata valores do ambiente afeta quais traits a closure implementa, e traits são como funções e structs podem especificar quais tipos de closures podem usar. Closures implementarão automaticamente uma, duas ou todas as três traits `Fn`, de forma aditiva, dependendo de como o corpo da closure trata os valores:

* `FnOnce` se aplica a closures que podem ser chamadas uma vez. Todas as closures implementam pelo menos esta trait porque todas as closures podem ser chamadas. Uma closure que move valores capturados para fora de seu corpo só implementará `FnOnce` e nenhuma das outras traits `Fn`, porque só pode ser chamada uma vez.
* `FnMut` se aplica a closures que não movem valores capturados para fora de seu corpo, mas podem mutar os valores capturados. Essas closures podem ser chamadas mais de uma vez.
* `Fn` se aplica a closures que não movem valores capturados para fora de seu corpo e não mutam valores capturados, bem como closures que não capturam nada de seu ambiente. Essas closures podem ser chamadas mais de uma vez sem mutar seu ambiente, o que é importante em casos como chamar uma closure várias vezes concorrentemente.

Vamos olhar a definição do método `unwrap_or_else` em `Option<T>` que usamos na Listagem 13-1:

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Lembre-se de que `T` é o tipo genérico representando o tipo do valor na variante `Some` de um `Option`. Esse tipo `T` também é o tipo de retorno da função `unwrap_or_else`: código que chama `unwrap_or_else` em um `Option<String>`, por exemplo, obterá um `String`.

Em seguida, note que a função `unwrap_or_else` tem o parâmetro de tipo genérico adicional `F`. O tipo `F` é o tipo do parâmetro chamado `f`, que é a closure que fornecemos ao chamar `unwrap_or_else`.

O trait bound especificado no tipo genérico `F` é `FnOnce() -> T`, o que significa que `F` deve poder ser chamado uma vez, não receber argumentos e retornar um `T`. Usar `FnOnce` no trait bound expressa a restrição de que `unwrap_or_else` não chamará `f` mais de uma vez. No corpo de `unwrap_or_else`, podemos ver que se o `Option` for `Some`, `f` não será chamado. Se o `Option` for `None`, `f` será chamado uma vez. Como todas as closures implementam `FnOnce`, `unwrap_or_else` aceita os três tipos de closures e é o mais flexível possível.

> **Nota:** Se o que queremos fazer não exige capturar um valor do ambiente, podemos usar o nome de uma função em vez de uma closure onde precisamos de algo que implemente uma das traits `Fn`. Por exemplo, em um valor `Option<Vec<T>>`, poderíamos chamar `unwrap_or_else(Vec::new)` para obter um novo vetor vazio se o valor for `None`. O compilador implementa automaticamente qualquer uma das traits `Fn` aplicável para uma definição de função.

Agora vamos olhar o método da biblioteca padrão `sort_by_key`, definido em slices, para ver como ele difere de `unwrap_or_else` e por que `sort_by_key` usa `FnMut` em vez de `FnOnce` como trait bound. A closure recebe um argumento na forma de uma referência ao item atual no slice sendo considerado e retorna um valor do tipo `K` que pode ser ordenado. Esta função é útil quando você quer ordenar um slice por um atributo particular de cada item. Na Listagem 13-7, temos uma lista de instâncias de `Rectangle`, e usamos `sort_by_key` para ordená-las pelo atributo `width`, do menor para o maior.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{list:#?}");
}
```

<a id="listagem-13-7"></a>

[Listagem 13-7](#listagem-13-7): Usando `sort_by_key` para ordenar retângulos por largura

Este código imprime:

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/rectangles`
[
    Rectangle {
        width: 3,
        height: 5,
    },
    Rectangle {
        width: 7,
        height: 12,
    },
    Rectangle {
        width: 10,
        height: 1,
    },
]
```

A razão pela qual `sort_by_key` é definido para receber uma closure `FnMut` é que ele chama a closure várias vezes: uma vez para cada item no slice. A closure `|r| r.width` não captura, muta nem move nada de seu ambiente, então atende aos requisitos do trait bound.

Em contraste, a Listagem 13-8 mostra um exemplo de closure que implementa apenas a trait `FnOnce`, porque move um valor para fora do ambiente. O compilador não nos deixará usar esta closure com `sort_by_key`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("closure called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{list:#?}");
}
```

<a id="listagem-13-8"></a>

[Listagem 13-8](#listagem-13-8): Tentativa de usar uma closure `FnOnce` com `sort_by_key`

Esta é uma forma contorcida e artificial (que não funciona) de tentar contar quantas vezes `sort_by_key` chama a closure ao ordenar `list`. Este código tenta fazer essa contagem empurrando `value`—um `String` do ambiente da closure—no vetor `sort_operations`. A closure captura `value` e então move `value` para fora da closure transferindo a posse de `value` para o vetor `sort_operations`. Esta closure pode ser chamada uma vez; tentar chamá-la uma segunda vez não funcionaria, porque `value` não estaria mais no ambiente para ser empurrado para `sort_operations` novamente! Portanto, esta closure só implementa `FnOnce`. Quando tentamos compilar este código, obtemos este erro de que `value` não pode ser movido para fora da closure porque a closure deve implementar `FnMut`:

```console
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
error[E0507]: cannot move out of `value`, a captured variable in an `FnMut` closure
  --> src/main.rs:18:30
   |
15 |     let value = String::from("closure called");
   |         -----   ------------------------------ move occurs because `value` has type `String`, which does not implement the `Copy` trait
   |         |
   |         captured outer variable
16 |
17 |     list.sort_by_key(|r| {
   |                      --- captured by this `FnMut` closure
18 |         sort_operations.push(value);
   |                              ^^^^^ `value` is moved here
   |
help: consider cloning the value if the performance cost is acceptable
   |
18 |         sort_operations.push(value.clone());
   |                                   ++++++++

For more information about this error, try `rustc --explain E0507`.
error: could not compile `rectangles` (bin "rectangles") due to 1 previous error
```

O erro aponta para a linha no corpo da closure que move `value` para fora do ambiente. Para corrigir isso, precisamos mudar o corpo da closure para que não mova valores para fora do ambiente. Manter um contador no ambiente e incrementar seu valor no corpo da closure é uma forma mais direta de contar quantas vezes a closure é chamada. A closure na Listagem 13-9 funciona com `sort_by_key` porque só captura uma referência mutável ao contador `num_sort_operations` e portanto pode ser chamada mais de uma vez.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{list:#?}, sorted in {num_sort_operations} operations");
}
```

<a id="listagem-13-9"></a>

[Listagem 13-9](#listagem-13-9): Usar uma closure `FnMut` com `sort_by_key` é permitido

As traits `Fn` são importantes ao definir ou usar funções ou tipos que fazem uso de closures. Na próxima seção, discutiremos iterators. Muitos métodos de iterator recebem closures como argumentos, então mantenha esses detalhes de closures em mente conforme continuamos!

[unwrap-or-else]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else
