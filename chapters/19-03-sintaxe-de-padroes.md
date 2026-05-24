---
title: "Sintaxe de padrões"
chapter_code: 19-03
slug: sintaxe-de-padroes
---

# Sintaxe de Padrões

Esta seção reúne toda a sintaxe válida em padrões e quando usá-la.

## Casando com literais

No Capítulo 6, padrões casam com literais diretamente:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Imprime `one` porque `x` é `1`. Útil quando você quer agir para um valor concreto.

## Casando com variáveis nomeadas

Variáveis nomeadas são padrões irrefutáveis que casam com qualquer valor. Em `match`, `if let` ou `while let`, cada expressão abre escopo novo; variáveis no padrão fazem shadow das externas.

A Listagem 19-11 declara `x = Some(5)` e `y = 10`, depois `match` em `x`. Tente prever a saída antes de executar.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {x:?}"),
    }

    println!("at the end: x = {x:?}, y = {y}");
}
```

[Listagem 19-11](#listagem-19-11): Braço de `match` que introduz `y` fazendo shadow de `y` existente

O primeiro braço não casa. O segundo introduz novo `y` que casa com qualquer valor em `Some` — liga a `5`. Imprime `Matched, y = 5`.

Se `x` fosse `None`, cairia no `_` sem introduzir `x` no padrão; imprimiria `Default case, x = None`. Ao fim do `match`, o `y` interno some; o último `println!` mostra `at the end: x = Some(5), y = 10`.

Para comparar `x` e `y` externos sem shadow, use match guard (Adicionando condicionais com match guards).

## Casando com vários padrões

Em `match`, use `|` (ou) para vários padrões:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Imprime `one or two`.

## Casando intervalos de valores com `..=`

`..=` casa intervalo inclusivo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
}
```

Se `x` for 1 a 5, o primeiro braço casa. Mais curto que `1 | 2 | 3 | 4 | 5`.

O compilador verifica intervalo não vazio em compile time; só `char` e numéricos permitem intervalos.

Com `char`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 'c';

    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
}
```

Imprime `early ASCII letter`.

## Desestruturando para separar valores

Padrões desestruturam structs, enums e tuplas.

### Structs

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

[Listagem 19-12](#listagem-19-12): Desestruturar campos de struct em variáveis separadas

Nomes no padrão não precisam coincidir com os campos. Forma abreviada (Listagem 19-13):

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

[Listagem 19-13](#listagem-19-13): Atalho de campos de struct no padrão

Podemos misturar literais no padrão (Listagem 19-14).

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

[Listagem 19-14](#listagem-19-14): Literais e desestruturação no mesmo padrão

`p` casa no segundo braço (`x = 0`); imprime `On the y axis at 7`. `match` para no primeiro braço que casa — `Point { x: 0, y: 0 }` seria eixo x, não ambos.

### Enums

Desestruturamos enums como na Listagem 6-5; o padrão segue a forma dos dados (Listagem 19-15).

**Arquivo: src/main.rs**

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
    }
}
```

[Listagem 19-15](#listagem-19-15): Desestruturar variantes de enum

Imprime `Change color to red 0, green 160, and blue 255`.

`Quit` só casa com o literal. `Move` usa chaves como struct. Tuplas em `Write` e `ChangeColor` exigem o mesmo número de variáveis.

### Structs e enums aninhados

**Arquivo: src/main.rs**

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}");
        }
        _ => (),
    }
}
```

[Listagem 19-16](#listagem-19-16): Casando enums aninhados

### Structs e tuplas

Desestruturação aninhada complexa:

**Arquivo: src/main.rs**

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
    }

    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
}
```

## Ignorando valores em um padrão

Às vezes ignoramos valores no último braço de `match` ou em partes do padrão: `_`, `_` aninhado, nome com `_` no início, ou `..`.

### Valor inteiro com `_`

`_` casa com qualquer valor sem ligar (Listagem 19-17).

**Arquivo: src/main.rs**

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {y}");
}

fn main() {
    foo(3, 4);
}
```

[Listagem 19-17](#listagem-19-17): `_` na assinatura de função

Útil ao implementar traits com parâmetros não usados — evita aviso de parâmetro morto.

### Partes de um valor com `_` aninhado

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }

    println!("setting is {setting_value:?}");
}
```

[Listagem 19-18](#listagem-19-18): `_` dentro de padrões `Some`

Imprime `Can't overwrite an existing customized value` e `setting is Some(5)`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}");
        }
    }
}
```

[Listagem 19-19](#listagem-19-19): Ignorar várias partes de uma tupla

Imprime `Some numbers: 2, 8, 32`.

### Variável não usada com nome começando em `_`

**Arquivo: src/main.rs**

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

[Listagem 19-20](#listagem-19-20): `_x` evita aviso; `y` ainda avisa

`_x` ainda liga o valor; `_` sozinho não liga. A Listagem 19-21 não compila:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let s = Some(String::from("Hello!"));

    if let Some(_s) = s {
        println!("found a string");
    }

    println!("{s:?}");
}
```

[Listagem 19-21](#listagem-19-21): `_s` ainda move `s`

` s` foi movido para `_s`. Com `_` sozinho (Listagem 19-22) compila:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }

    println!("{s:?}");
}
```

[Listagem 19-22](#listagem-19-22): `_` não liga o valor

### Restante do valor com `..`

**Arquivo: src/main.rs**

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }

    let origin = Point { x: 0, y: 0, z: 0 };

    match origin {
        Point { x, .. } => println!("x is {x}"),
    }
}
```

[Listagem 19-23](#listagem-19-23): `..` ignora `y` e `z`

Em tuplas (Listagem 19-24):

**Arquivo: src/main.rs**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

[Listagem 19-24](#listagem-19-24): Primeiro e último da tupla, ignorando o meio

`..` deve ser inequívoco. A Listagem 19-25 não compila:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {second}")
        },
    }
}
```

[Listagem 19-25](#listagem-19-25): Uso ambíguo de `..`

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

## Adicionando condicionais com match guards

_match guard_ é um `if` extra após o padrão no braço de `match`. Só em `match`, não em `if let` ou `while let`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {x} is even"),
        Some(x) => println!("The number {x} is odd"),
        None => (),
    }
}
```

[Listagem 19-26](#listagem-19-26): Match guard em um padrão

Imprime `The number 4 is even`. Com `Some(5)`, o guard do primeiro braço falha e o segundo casa.

O compilador não verifica exaustividade com guards.

A Listagem 19-27 resolve o shadow da Listagem 19-11:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {x:?}"),
    }

    println!("at the end: x = {x:?}, y = {y}");
}
```

[Listagem 19-27](#listagem-19-27): Guard para comparar com variável externa

Imprime `Default case, x = Some(5)`. `Some(n)` não faz shadow de `y`; `if n == y` usa o `y` externo.

`|` com guard: o `if` vale para todos os padrões do braço (Listagem 19-28).

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
}
```

[Listagem 19-28](#listagem-19-28): Vários padrões com um match guard

Imprime `no` — precedência é `(4 | 5 | 6) if y`, não `4 | 5 | (6 if y)`.

## Usando bindings `@`

`@` cria variável com o valor enquanto testa o padrão (Listagem 19-29).

**Arquivo: src/main.rs**

```rust
fn main() {
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello { id: id @ 3..=7 } => {
            println!("Found an id in range: {id}")
        }
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {id}"),
    }
}
```

[Listagem 19-29](#listagem-19-29): `@` para ligar e testar ao mesmo tempo

Imprime `Found an id in range: 5`. No segundo braço, o intervalo casa mas não há variável `id` no código do braço. No terceiro, `id` está disponível sem teste de intervalo.

## Resumo

Padrões distinguem tipos de dados. Em `match`, Rust exige cobrir todos os valores. Em `let` e parâmetros, desestruturam valores em partes menores.

No penúltimo capítulo do livro, veremos aspectos avançados de vários recursos do Rust.
