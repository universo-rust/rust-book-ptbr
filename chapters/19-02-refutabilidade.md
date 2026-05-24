---
title: "Refutabilidade: se um padrão pode falhar ao casar"
chapter_code: 19-02
slug: refutabilidade-se-um-padrao-pode-falhar-ao-casar
---

# Refutabilidade: Se um Padrão Pode Falhar ao Casar

Padrões têm duas formas: refutáveis e irrefutáveis. Padrões que casam com qualquer valor possível são _irrefutáveis_ — por exemplo `x` em `let x = 5;`, porque `x` casa com tudo e não pode falhar.

Padrões que podem falhar para algum valor são _refutáveis_ — por exemplo `Some(x)` em `if let Some(x) = a_value`, porque se `a_value` for `None`, `Some(x)` não casa.

Parâmetros de função, `let` e loops `for` só aceitam padrões irrefutáveis, porque o programa não tem o que fazer de útil quando o valor não casa. `if let`, `while let` e `let...else` aceitam refutáveis e irrefutáveis, mas o compilador avisa com padrões irrefutáveis — a ideia é lidar com sucesso ou falha.

Em geral não precisa decorar a distinção, mas precisa reconhecê-la em mensagens de erro e ajustar padrão ou construção conforme o comportamento desejado.

A Listagem 19-8 usa padrão refutável `Some(x)` em `let`, que não compila.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value;
}
```

[Listagem 19-8](#listagem-19-8): Tentativa de usar padrão refutável com `let`

Se `some_option_value` for `None`, não casa com `Some(x)` — refutável. `let` só aceita irrefutável porque não há o que fazer com `None`:

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

Como `Some(x)` não cobre todos os valores, o compilador acusa erro.

Use `let...else` em vez de `let` quando precisar de refutável onde se exige irrefutável (Listagem 19-9).

**Arquivo: src/main.rs**

```rust
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value else {
        return;
    };
}
```

[Listagem 19-9](#listagem-19-9): `let...else` com bloco para padrões refutáveis

Código válido. `let...else` com padrão que sempre casa, como `x`, gera aviso (Listagem 19-10).

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5 else {
        return;
    };
}
```

[Listagem 19-10](#listagem-19-10): Tentativa de padrão irrefutável com `let...else`

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

Braços de `match` devem ser refutáveis, exceto o último, que deve ser irrefutável (catch-all). Rust permite irrefutável em `match` de um braço só, mas `let` seria mais simples.

Agora cobrimos onde usar padrões e refutabilidade; em seguida, toda a sintaxe de padrões.
