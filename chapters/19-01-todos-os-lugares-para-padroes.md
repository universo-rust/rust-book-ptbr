---
title: "Todos os lugares onde padrões podem ser usados"
chapter_code: 19-01
slug: todos-os-lugares-onde-padroes-podem-ser-usados
---

# Todos os Lugares Onde Padrões Podem Ser Usados

Padrões aparecem em vários lugares em Rust, e você já os usou bastante sem perceber. Esta seção discute todos os lugares válidos.

## Braços de `match`

Como no Capítulo 6, usamos padrões nos braços de `match`. Formalmente, `match` é a palavra-chave `match`, um valor a casar e um ou mais braços com padrão e expressão a executar se o valor casar, assim:

```
match VALOR {
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
}
```

Por exemplo, o `match` da Listagem 6-5 sobre `Option<i32>` em `x`:

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Os padrões são `None` e `Some(i)` à esquerda de cada seta.

`match` precisa ser exaustivo: todas as possibilidades do valor devem ser cobertas. Um braço catch-all no final ajuda — um nome de variável que casa com qualquer valor nunca falha.

O padrão `_` casa com qualquer coisa, mas não liga a variável; costuma ir no último braço para ignorar o restante. Veremos `_` em Ignorando valores em um padrão.

## Instruções `let`

Além de `match` e `if let`, usamos padrões em `let`:

```rust
let x = 5;
```

Cada `let` assim usa padrões. Formalmente:

```
let PADRÃO = EXPRESSÃO;
```

Em `let x = 5;`, `x` é um padrão simples: “ligue o que casar aqui à variável `x`”.

A Listagem 19-1 desestrutura uma tupla com `let`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let (x, y, z) = (1, 2, 3);
}
```

[Listagem 19-1](#listagem-19-1): Padrão para desestruturar tupla e criar três variáveis

Rust compara `(1, 2, 3)` ao padrão `(x, y, z)`, vê que o número de elementos coincide e liga `1` a `x`, `2` a `y`, `3` a `z`.

Se o número de elementos não coincidir, há erro de compilação (Listagem 19-2).

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let (x, y) = (1, 2, 3);
}
```

[Listagem 19-2](#listagem-19-2): Padrão com número de variáveis diferente do da tupla

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

Podemos ignorar valores com `_` ou `..` (ver Ignorando valores em um padrão) ou remover variáveis até o número coincidir.

## Expressões condicionais `if let`

No Capítulo 6, `if let` encurta um `match` de um único caso. Pode ter `else` se o padrão não casar.

A Listagem 19-3 mistura `if let`, `else if` e `else if let` — mais flexível que um `match` com um só valor; os braços não precisam estar relacionados.

**Arquivo: src/main.rs**

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

[Listagem 19-3](#listagem-19-3): Misturando `if let`, `else if`, `else if let` e `else`

Com os valores fixos do exemplo, imprime `Using purple as the background color`.

`if let` pode introduzir variáveis que fazem shadow, como braços de `match`: `if let Ok(age) = age` cria novo `age` dentro do `Ok`. Não dá para juntar em `if let Ok(age) = age && age > 30` — o novo `age` só existe após o `{`.

Desvantagem: o compilador não verifica exaustividade, diferente de `match`. Sem o `else` final, casos omitidos não geram aviso.

## Loops condicionais `while let`

Como `if let`, `while let` roda enquanto o padrão continuar casando. A Listagem 19-4 espera mensagens em um channel, tratando `Result` em vez de `Option`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let (tx, rx) = std::sync::mpsc::channel();
    std::thread::spawn(move || {
        for val in [1, 2, 3] {
            tx.send(val).unwrap();
        }
    });

    while let Ok(value) = rx.recv() {
        println!("{value}");
    }
}
```

[Listagem 19-4](#listagem-19-4): `while let` enquanto `rx.recv()` retorna `Ok`

Imprime `1`, `2`, `3`. `recv` tira a primeira mensagem e retorna `Ok(value)` enquanto o sender existir; depois `Err` quando desconecta. No Capítulo 16 vimos `unwrap` ou `for`; aqui `while let` também serve.

## Loops `for`

Em `for`, o valor após `for` é um padrão. Em `for x in y`, `x` é o padrão. A Listagem 19-5 desestrutura tuplas no `for`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{value} is at index {index}");
    }
}
```

[Listagem 19-5](#listagem-19-5): Padrão em `for` para desestruturar tupla

Saída:

```text
a is at index 0
b is at index 1
c is at index 2
```

`enumerate` produz `(0, 'a')` etc.; o padrão `(index, value)` liga os nomes.

## Parâmetros de função

Parâmetros de função também são padrões. A Listagem 19-6 declara `foo` com parâmetro `x: i32`.

**Arquivo: src/main.rs**

```rust
fn foo(x: i32) {
    // o código vai aqui
}

fn main() {}
```

[Listagem 19-6](#listagem-19-6): Assinatura de função com padrões nos parâmetros

`x` é um padrão. Podemos casar tupla nos argumentos (Listagem 19-7).

**Arquivo: src/main.rs**

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({x}, {y})");
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

[Listagem 19-7](#listagem-19-7): Função que desestrutura tupla nos parâmetros

Imprime `Current location: (3, 5)`. `&(3, 5)` casa com `&(x, y)`.

O mesmo vale para parâmetros de closures (Capítulo 13).

Até aqui vimos vários usos de padrões, mas não funcionam igual em todo lugar: em alguns só padrões irrefutáveis; em outros, refutáveis. Isso vem a seguir.
