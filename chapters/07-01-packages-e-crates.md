---
title: "Packages e crates"
chapter_code: 07-01
slug: packages-e-crates
---

# Packages e Crates

As primeiras partes do sistema de módulos que cobriremos são packages e crates.

Um _crate_ é a menor quantidade de código que o compilador Rust considera de uma vez. Mesmo se você executar `rustc` em vez de `cargo` e passar um único arquivo de código-fonte (como fizemos lá no início no Capítulo 1), o compilador considera esse arquivo um crate. Crates podem conter módulos, e os módulos podem ser definidos em outros arquivos que são compilados com o crate, como veremos nas próximas seções.

Um crate pode ter uma de duas formas: um binary crate ou um library crate. _Binary crates_ são programas que você pode compilar em um executável que pode executar, como um programa de linha de comando ou um servidor. Cada um deve ter uma função chamada `main` que define o que acontece quando o executável roda. Todos os crates que criamos até agora foram binary crates.

_Library crates_ não têm função `main` e não compilam para um executável. Em vez disso, definem funcionalidade destinada a ser compartilhada com vários projetos. Por exemplo, o crate `rand` que usamos no Capítulo 2 fornece funcionalidade que gera números aleatórios. Na maioria das vezes, quando rustaceans dizem “crate”, querem dizer library crate, e usam “crate” de forma intercambiável com o conceito geral de programação de “biblioteca”.

A _raiz do crate_ (_crate root_) é um arquivo-fonte do qual o compilador Rust começa e que forma o módulo raiz do seu crate (explicaremos módulos em profundidade em Controlar escopo e privacidade com módulos).

Um _package_ é um pacote de um ou mais crates que fornece um conjunto de funcionalidades. Um package contém um arquivo _Cargo.toml_ que descreve como compilar esses crates. O Cargo é, na verdade, um package que contém o binary crate da ferramenta de linha de comando que você tem usado para compilar seu código. O package do Cargo também contém um library crate do qual o binary crate depende. Outros projetos podem depender do library crate do Cargo para usar a mesma lógica que a ferramenta de linha de comando do Cargo usa.

Um package pode conter quantos binary crates quiser, mas no máximo apenas um library crate. Um package deve conter pelo menos um crate, seja library ou binary.

Vamos ver o que acontece quando criamos um package. Primeiro, digitamos o comando `cargo new my-project`:

```bash
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Depois de executar `cargo new my-project`, usamos `ls` para ver o que o Cargo cria. No diretório _my-project_, há um arquivo _Cargo.toml_, nos dando um package. Há também um diretório _src_ que contém _main.rs_. Abra _Cargo.toml_ no seu editor de texto e note que não há menção a _src/main.rs_. O Cargo segue uma convenção de que _src/main.rs_ é a raiz do crate de um binary crate com o mesmo nome do package. Da mesma forma, o Cargo sabe que, se o diretório do package contiver _src/lib.rs_, o package contém um library crate com o mesmo nome do package, e _src/lib.rs_ é sua raiz do crate. O Cargo passa os arquivos de raiz do crate para `rustc` para compilar a biblioteca ou o binário.

Aqui, temos um package que só contém _src/main.rs_, o que significa que contém apenas um binary crate chamado `my-project`. Se um package contiver _src/main.rs_ e _src/lib.rs_, terá dois crates: um binário e uma biblioteca, ambos com o mesmo nome do package. Um package pode ter vários binary crates colocando arquivos no diretório _src/bin_: cada arquivo será um binary crate separado.
