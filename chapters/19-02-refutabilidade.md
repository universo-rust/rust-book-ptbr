---
title: "Refutabilidade: se um padrão pode falhar ao casar"
chapter_code: 19-02
slug: refutabilidade-se-um-padrao-pode-falhar-ao-casar
---

# Refutabilidade: se um padrão pode falhar ao casar

Padrões existem em duas formas: refutáveis e irrefutáveis. Padrões que casam com qualquer valor possível recebido são _irrefutáveis_. Um exemplo é `x` na instrução `let x = 5;`, porque `x` casa com qualquer coisa e, portanto, não pode falhar ao casar. Padrões que podem falhar para algum valor possível são _refutáveis_. Um exemplo é `Some(x)` na expressão `if let Some(x) = a_value`, porque, se o valor na variável `a_value` for `None` em vez de `Some`, o padrão `Some(x)` não casará.

Parâmetros de função, instruções `let` e loops `for` só podem aceitar padrões irrefutáveis, porque o programa não teria nada significativo a fazer quando os valores não casassem. Expressões `if let` e `while let`, assim como instruções `let...else`, aceitam padrões refutáveis e irrefutáveis, mas o compilador avisa contra padrões irrefutáveis nesses contextos porque, por definição, eles foram feitos para lidar com uma possível falha. A utilidade de uma condicional está justamente em poder agir de forma diferente dependendo do sucesso ou da falha.

Em geral, você não precisa se preocupar muito com a diferença entre padrões refutáveis e irrefutáveis. No entanto, precisa estar familiarizado com o conceito de refutabilidade para saber como responder quando ele aparecer em uma mensagem de erro. Nesses casos, você precisará mudar o padrão ou a construção em que está usando o padrão, dependendo do comportamento pretendido para o código.

Vamos ver o que acontece quando tentamos usar um padrão refutável onde Rust exige um padrão irrefutável, e vice-versa. A Listagem 19-8 mostra uma instrução `let`, mas usamos `Some(x)` como padrão, que é refutável. Como você talvez espere, esse código não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value;
}
```

<a id="listagem-19-8"></a>

[Listagem 19-8](#listagem-19-8): Tentativa de usar um padrão refutável com `let`

Se `some_option_value` fosse um valor `None`, ele não casaria com o padrão `Some(x)`, o que significa que o padrão é refutável. Porém, a instrução `let` só pode aceitar um padrão irrefutável, porque não há nada válido que o código possa fazer com um valor `None`. Em tempo de compilação, Rust reclamará que tentamos usar um padrão refutável onde um padrão irrefutável é exigido:

```console
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch19-02-refutability.html
  = note: the matched value is of type `Option<i32>`
help: you might want to use `let else` to handle the variant that isn't matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

Como não cobrimos, e nem poderíamos cobrir, todos os valores válidos com o padrão `Some(x)`, Rust corretamente produz um erro de compilação.

Se temos um padrão refutável onde um padrão irrefutável é necessário, podemos corrigir o problema mudando a construção que usa o padrão. Em vez de `let`, podemos usar `let...else`. Assim, se o padrão não casar, o código entre chaves tratará o valor. A Listagem 19-9 mostra como corrigir o código da Listagem 19-8.

**Arquivo: src/main.rs**

```rust
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value else {
        return;
    };
}
```

<a id="listagem-19-9"></a>

[Listagem 19-9](#listagem-19-9): Usando `let...else` e um bloco com padrões refutáveis em vez de `let`

Demos uma saída ao código! Esse código é perfeitamente válido, embora isso também signifique que não podemos usar um padrão irrefutável sem receber um aviso. Se passarmos a `let...else` um padrão que sempre casa, como `x`, como mostra a Listagem 19-10, o compilador emitirá um aviso.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5 else {
        return;
    };
}
```

<a id="listagem-19-10"></a>

[Listagem 19-10](#listagem-19-10): Tentativa de usar um padrão irrefutável com `let...else`

Rust reclama que não faz sentido usar `let...else` com um padrão irrefutável:

```console
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `let...else` pattern
 --> src/main.rs:2:5
  |
2 |     let x = 5 else {
  |     ^^^^^^^^^
  |
  = note: this pattern will always match, so the `else` clause is useless
  = help: consider removing the `else` clause
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin "patterns") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
```

Por essa razão, braços de `match` devem usar padrões refutáveis, exceto o último braço, que deve casar com quaisquer valores restantes usando um padrão irrefutável. Rust permite usar um padrão irrefutável em um `match` com apenas um braço, mas essa sintaxe não é particularmente útil e poderia ser substituída por uma instrução `let` mais simples.

Agora que você sabe onde usar padrões e entende a diferença entre padrões refutáveis e irrefutáveis, vamos cobrir toda a sintaxe que podemos usar para criar padrões.
