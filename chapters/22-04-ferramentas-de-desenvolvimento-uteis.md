---
title: "Ferramentas de desenvolvimento úteis"
chapter_code: 22-04
slug: ferramentas-de-desenvolvimento-uteis
---

# Apêndice D: Ferramentas de desenvolvimento úteis

Neste apêndice, falamos sobre algumas ferramentas de desenvolvimento úteis que o projeto Rust fornece. Veremos formatação automática, maneiras rápidas de aplicar correções de avisos, um linter e integração com IDEs.

### Formatação automática com `rustfmt`

A ferramenta `rustfmt` reformata seu código de acordo com o estilo de código da comunidade. Muitos projetos colaborativos usam `rustfmt` para evitar discussões sobre qual estilo usar ao escrever Rust: todos formatam seu código usando a ferramenta.

As instalações do Rust incluem `rustfmt` por padrão, então você já deve ter os programas `rustfmt` e `cargo-fmt` no seu sistema. Esses dois comandos são análogos a `rustc` e `cargo` no sentido de que `rustfmt` permite controle mais fino e `cargo-fmt` entende as convenções de um projeto que usa Cargo. Para formatar qualquer projeto Cargo, digite o seguinte:

```console
$ cargo fmt
```

Executar este comando reformata todo o código Rust no crate atual. Isso deve alterar apenas o estilo do código, não a semântica do código. Para mais informações sobre `rustfmt`, consulte [sua documentação](https://github.com/rust-lang/rustfmt).

### Corrija seu código com `rustfix`

A ferramenta `rustfix` vem com as instalações do Rust e pode corrigir automaticamente avisos do compilador que tenham uma forma clara de corrigir o problema que provavelmente é o que você deseja. Você provavelmente já viu avisos do compilador antes. Por exemplo, considere este código:

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

Aqui, estamos definindo a variável `x` como mutável, mas nunca a mutamos de fato. O Rust nos avisa sobre isso:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

O aviso sugere que removamos a palavra-chave `mut`. Podemos aplicar automaticamente essa sugestão usando a ferramenta `rustfix` executando o comando `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Quando olharmos _src/main.rs_ novamente, veremos que `cargo fix` alterou o código:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

A variável `x` agora é imutável, e o aviso não aparece mais.

Você também pode usar o comando `cargo fix` para migrar seu código entre diferentes edições do Rust. As edições são abordadas no [Apêndice E](/livro/cap22-05-edicoes).

### Mais lints com o Clippy

A ferramenta Clippy é uma coleção de lints para analisar seu código para que você possa detectar erros comuns e melhorar seu código Rust. O Clippy está incluído nas instalações padrão do Rust.

Para executar os lints do Clippy em qualquer projeto Cargo, digite o seguinte:

```console
$ cargo clippy
```

Por exemplo, digamos que você escreva um programa que usa uma aproximação de uma constante matemática, como pi, como este programa faz:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Executar `cargo clippy` neste projeto resulta neste erro:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Este erro informa que o Rust já tem uma constante `PI` mais precisa definida e que seu programa seria mais correto se você usasse a constante em vez disso. Você então alteraria seu código para usar a constante `PI`.

O código a seguir não resulta em erros ou avisos do Clippy:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Para mais informações sobre o Clippy, consulte [sua documentação](https://github.com/rust-lang/rust-clippy).

### Integração com IDE usando `rust-analyzer`

Para ajudar na integração com IDEs, a comunidade Rust recomenda usar [`rust-analyzer`](https://rust-analyzer.github.io). Esta ferramenta é um conjunto de utilitários centrados no compilador que falam o [Language Server Protocol](http://langserver.org/), que é uma especificação para IDEs e linguagens de programação se comunicarem entre si. Diferentes clientes podem usar o `rust-analyzer`, como [o plug-in Rust Analyzer para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer).

Visite a [página inicial](https://rust-analyzer.github.io) do projeto `rust-analyzer` para instruções de instalação e, em seguida, instale o suporte ao language server na sua IDE específica. Sua IDE ganhará recursos como autocompletar, ir para definição e erros inline.
