---
title: "Funções e closures avançadas"
chapter_code: 20-04
slug: funcoes-e-closures-avancadas
---

# Funções e closures avançadas

Esta seção explora alguns recursos avançados relacionados a funções e closures, incluindo ponteiros de função e retorno de closures.

## Ponteiros de função

Já falamos sobre como passar closures para funções; você também pode passar funções regulares para funções! Essa técnica é útil quando você quer passar uma função que já definiu em vez de definir uma nova closure. Funções sofrem coerção para o tipo `fn` (com _f_ minúsculo), que não deve ser confundido com a trait de closure `Fn`. O tipo `fn` é chamado de _ponteiro de função_. Passar funções com ponteiros de função permitirá que você use funções como argumentos para outras funções.

A sintaxe para especificar que um parâmetro é um ponteiro de função é semelhante à de closures, como mostrado na Listagem 20-28, onde definimos uma função `add_one` que adiciona 1 ao seu parâmetro. A função `do_twice` recebe dois parâmetros: um ponteiro de função para qualquer função que receba um parâmetro `i32` e retorne um `i32`, e um valor `i32`. A função `do_twice` chama a função `f` duas vezes, passando a ela o valor `arg`, e então soma os dois resultados das chamadas de função. A função `main` chama `do_twice` com os argumentos `add_one` e `5`.

**Arquivo: src/main.rs**

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {answer}");
}
```

<a id="listagem-20-28"></a>

[Listagem 20-28](#listagem-20-28): Usando o tipo `fn` para aceitar um ponteiro de função como argumento

Este código imprime `The answer is: 12`. Especificamos que o parâmetro `f` em `do_twice` é um `fn` que recebe um parâmetro do tipo `i32` e retorna um `i32`. Podemos então chamar `f` no corpo de `do_twice`. Em `main`, podemos passar o nome da função `add_one` como o primeiro argumento para `do_twice`.

Diferente de closures, `fn` é um tipo em vez de uma trait, então especificamos `fn` como o tipo do parâmetro diretamente em vez de declarar um parâmetro de tipo genérico com uma das traits `Fn` como trait bound.

Ponteiros de função implementam as três traits de closure (`Fn`, `FnMut` e `FnOnce`), o que significa que você sempre pode passar um ponteiro de função como argumento para uma função que espera uma closure. É melhor escrever funções usando um tipo genérico e uma das traits de closure para que suas funções possam aceitar funções ou closures.

Dito isso, um exemplo de situação em que você quereria aceitar apenas `fn` e não closures é ao interagir com código externo que não tem closures: funções C podem aceitar funções como argumentos, mas C não tem closures.

Como exemplo de situação em que você poderia usar uma closure definida inline ou uma função nomeada, vamos olhar um uso do método `map` fornecido pela trait `Iterator` na biblioteca padrão. Para usar o método `map` para transformar um vetor de números em um vetor de strings, poderíamos usar uma closure, como na Listagem 20-29.

**Arquivo: src/main.rs**

```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
}
```

<a id="listagem-20-29"></a>

[Listagem 20-29](#listagem-20-29): Usando uma closure com o método `map` para converter números em strings

Ou poderíamos nomear uma função como argumento de `map` em vez da closure. A Listagem 20-30 mostra como isso ficaria.

**Arquivo: src/main.rs**

```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
}
```

<a id="listagem-20-30"></a>

[Listagem 20-30](#listagem-20-30): Usando a função `String::to_string` com o método `map` para converter números em strings

Observe que devemos usar a sintaxe totalmente qualificada que discutimos na seção Traits avançadas do Capítulo 20 porque há múltiplas funções disponíveis chamadas `to_string`.

Aqui, estamos usando a função `to_string` definida na trait `ToString`, que a biblioteca padrão implementou para qualquer tipo que implemente `Display`.

Lembre-se da seção Valores de enum no Capítulo 6 de que o nome de cada variante de enum que definimos também se torna uma função inicializadora. Podemos usar essas funções inicializadoras como ponteiros de função que implementam as traits de closure, o que significa que podemos especificar as funções inicializadoras como argumentos para métodos que recebem closures, como visto na Listagem 20-31.

**Arquivo: src/main.rs**

```rust
fn main() {
    enum Status {
        Value(u32),
        Stop,
    }

    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
}
```

<a id="listagem-20-31"></a>

[Listagem 20-31](#listagem-20-31): Usando um inicializador de enum com o método `map` para criar uma instância de `Status` a partir de números

Aqui, criamos instâncias de `Status::Value` usando cada valor `u32` no intervalo sobre o qual `map` é chamado, usando a função inicializadora de `Status::Value`. Algumas pessoas preferem este estilo e outras preferem usar closures. Elas compilam para o mesmo código, então use o estilo que for mais claro para você.

## Retornando closures

Closures são representadas por traits, o que significa que você não pode retornar closures diretamente. Na maioria dos casos em que você pode querer retornar uma trait, pode em vez disso usar o tipo concreto que implementa a trait como valor de retorno da função. Porém, você normalmente não pode fazer isso com closures porque elas não têm um tipo concreto que seja retornável; você não tem permissão para usar o ponteiro de função `fn` como tipo de retorno se a closure capturar quaisquer valores de seu escopo, por exemplo.

Em vez disso, você normalmente usará a sintaxe `impl Trait` que aprendemos no Capítulo 10. Você pode retornar qualquer tipo de função, usando `Fn`, `FnOnce` e `FnMut`. Por exemplo, o código na Listagem 20-32 compilará sem problemas.

**Arquivo: src/lib.rs**

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

<a id="listagem-20-32"></a>

[Listagem 20-32](#listagem-20-32): Retornando uma closure de uma função usando a sintaxe `impl Trait`

Porém, como observamos na seção Inferindo e Anotando Tipos de Closure do Capítulo 13, cada closure também é seu próprio tipo distinto. Se você precisar trabalhar com múltiplas funções que têm a mesma assinatura, mas implementações diferentes, precisará usar um trait object para elas. Considere o que acontece se você escrever código como o mostrado na Listagem 20-33.

**Arquivo: src/main.rs**

```rust,ignore,does_not_compile
fn main() {
    let handlers = vec![returns_closure(), returns_initialized_closure(123)];
    for handler in handlers {
        let output = handler(5);
        println!("{output}");
    }
}

fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}

fn returns_initialized_closure(init: i32) -> impl Fn(i32) -> i32 {
    move |x| x + init
}
```

<a id="listagem-20-33"></a>

[Listagem 20-33](#listagem-20-33): Criando um `Vec<T>` de closures definidas por funções que retornam tipos `impl Fn`

Aqui temos duas funções, `returns_closure` e `returns_initialized_closure`, que ambas retornam `impl Fn(i32) -> i32`. Observe que as closures que elas retornam são diferentes, embora implementem o mesmo tipo. Se tentarmos compilar isso, Rust nos informa que não funcionará:

```text
$ cargo build
   Compiling functions-example v0.1.0 (file:///projects/functions-example)
error[E0308]: mismatched types
  --> src/main.rs:2:44
   |
 2 |     let handlers = vec![returns_closure(), returns_initialized_closure(123)];
   |                                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected opaque type, found a different opaque type
...
 9 | fn returns_closure() -> impl Fn(i32) -> i32 {
   |                         ------------------- the expected opaque type
...
13 | fn returns_initialized_closure(init: i32) -> impl Fn(i32) -> i32 {
   |                                              ------------------- the found opaque type
   |
   = note: expected opaque type `impl Fn(i32) -> i32`
              found opaque type `impl Fn(i32) -> i32`
   = note: distinct uses of `impl Trait` result in different opaque types

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions-example` (bin "functions-example") due to 1 previous error
```

A mensagem de erro nos diz que sempre que retornamos um `impl Trait`, Rust cria um _tipo opaco_ único, um tipo no qual não podemos ver os detalhes do que Rust constrói para nós, nem podemos adivinhar o tipo que Rust gerará para escrever nós mesmos. Então, embora essas funções retornem closures que implementam a mesma trait, `Fn(i32) -> i32`, os tipos opacos que Rust gera para cada uma são distintos. (Isso é semelhante a como Rust produz tipos concretos diferentes para blocos async distintos mesmo quando eles têm o mesmo tipo de saída, como vimos em O tipo `Pin` e a trait `Unpin` no Capítulo 17.) Já vimos uma solução para este problema algumas vezes agora: podemos usar um trait object, como na Listagem 20-34.

**Arquivo: src/main.rs**

```rust
fn main() {
    let handlers = vec![
        returns_closure(),
        returns_initialized_closure(123),
    ];
    for handler in handlers {
        let output = handler(5);
        println!("{output}");
    }
}

fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}

fn returns_initialized_closure(init: i32) -> Box<dyn Fn(i32) -> i32> {
    Box::new(move |x| x + init)
}
```

<a id="listagem-20-34"></a>

[Listagem 20-34](#listagem-20-34): Criando um `Vec<T>` de closures definidas por funções que retornam `Box<dyn Fn>` para que tenham o mesmo tipo

Este código compilará sem problemas. Para mais sobre trait objects, consulte a seção [Usando trait objects para abstrair comportamento compartilhado](/livro/cap18-02-usando-trait-objects-para-abstrair-comportamento-compartilhado) do Capítulo 18.

Em seguida, vamos olhar macros!
