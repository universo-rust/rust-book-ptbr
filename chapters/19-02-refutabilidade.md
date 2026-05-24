---
title: "Refutabilidade: se um padrão pode falhar ao casar"
chapter_code: 19-02
slug: refutabilidade-se-um-padrao-pode-falhar-ao-casar
---

# Refutabilidade: Se um Padrão Pode Falhar ao Casar

Padrões vêm em duas formas: refutáveis e irrefutáveis. Padrões que casarão com qualquer valor possível passado são _irrefutáveis_. Um exemplo seria `x` na instrução `let x = 5;`, porque `x` casa com qualquer coisa e, portanto, não pode falhar ao casar. Padrões que podem falhar ao casar para algum valor possível são _refutáveis_. Um exemplo seria `Some(x)` na expressão `if let Some(x) = a_value`, porque se o valor na variável `a_value` for `None` em vez de `Some`, o padrão `Some(x)` não casará.

Parâmetros de função, instruções `let` e loops `for` só podem aceitar padrões irrefutáveis porque o programa não pode fazer nada significativo quando os valores não casam. As expressões `if let` e `while let` e a instrução `let...else` aceitam padrões refutáveis e irrefutáveis, mas o compilador avisa contra padrões irrefutáveis porque, por definição, elas são destinadas a lidar com possível falha: A funcionalidade de uma condicional está em sua capacidade de se comportar de forma diferente dependendo do sucesso ou da falha.

Em geral, você não deveria precisar se preocupar com a distinção entre padrões refutáveis e irrefutáveis; no entanto, você precisa estar familiarizado com o conceito de refutabilidade para poder responder quando vê-lo em uma mensagem de erro. Nesses casos, você precisará mudar ou o padrão ou a construção com a qual está usando o padrão, dependendo do comportamento pretendido do código.

Vamos ver um exemplo do que acontece quando tentamos usar um padrão refutável onde Rust exige um padrão irrefutável e vice-versa. A Listagem 19-8 mostra uma instrução `let`, mas para o padrão, especificamos `Some(x)`, um padrão refutável. Como você pode esperar, este código não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value;
}
```

<a id="listagem-19-8"></a>

[Listagem 19-8](#listagem-19-8): Tentativa de usar um padrão refutável com `let`

Se `some_option_value` fosse um valor `None`, ele falharia ao casar com o padrão `Some(x)`, o que significa que o padrão é refutável. No entanto, a instrução `let` só pode aceitar um padrão irrefutável porque não há nada válido que o código possa fazer com um valor `None`. Em tempo de compilação, Rust reclamará que tentamos usar um padrão refutável onde um padrão irrefutável é exigido:

```bash
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

Como não cobrimos (e não poderíamos cobrir!) todo valor válido com o padrão `Some(x)`, Rust corretamente produz um erro de compilador.

Se temos um padrão refutável onde um padrão irrefutável é necessário, podemos corrigir mudando o código que usa o padrão: Em vez de usar `let`, podemos usar `let...else`. Então, se o padrão não casar, o código entre chaves tratará o valor. A Listagem 19-9 mostra como corrigir o código da Listagem 19-8.

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

Demos uma saída ao código! Este código é perfeitamente válido, embora signifique que não podemos usar um padrão irrefutável sem receber um aviso. Se dermos a `let...else` um padrão que sempre casará, como `x`, como mostrado na Listagem 19-10, o compilador emitirá um aviso.

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

```bash
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

Por essa razão, braços de match devem usar padrões refutáveis, exceto o último braço, que deve casar com quaisquer valores restantes com um padrão irrefutável. Rust nos permite usar um padrão irrefutável em um `match` com apenas um braço, mas esta sintaxe não é particularmente útil e poderia ser substituída por uma instrução `let` mais simples.

Agora que você sabe onde usar padrões e a diferença entre padrões refutáveis e irrefutáveis, vamos cobrir toda a sintaxe que podemos usar para criar padrões.
