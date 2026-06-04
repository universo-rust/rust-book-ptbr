---
title: "Packages e crates"
chapter_code: 07-01
slug: packages-e-crates
challenge_day: 8
reading_minutes: 6
---

# Packages e crates

As primeiras partes do sistema de módulos que vamos abordar são packages e crates.

Um _crate_ é a menor unidade de código que o compilador Rust considera de cada vez. Mesmo se você executar `rustc` em vez de `cargo` e passar um único arquivo-fonte (como fizemos lá em [Fundamentos de Programas em Rust](/livro/cap01-02-hello-world#noções-básicas-de-um-programa-rust), no Capítulo 1), o compilador considera esse arquivo um crate. Crates podem conter módulos, e esses módulos podem ser definidos em outros arquivos compilados junto com o crate, como veremos nas próximas seções.

Um crate pode ter uma de duas formas: um binary crate ou um library crate. _Binary crates_ são programas que você pode compilar em um executável e rodar, como um programa de linha de comando ou um servidor. Cada um deles precisa ter uma função chamada `main`, que define o que acontece quando o executável é iniciado. Todos os crates que criamos até agora foram binary crates.

_Library crates_ não têm função `main` e não são compilados como executáveis. Em vez disso, definem funcionalidades pensadas para serem compartilhadas entre vários projetos. Por exemplo, o crate `rand`, que usamos no Capítulo 2, oferece funcionalidades para gerar números aleatórios. Na maior parte do tempo, quando rustáceos dizem "crate", estão se referindo a library crate, usando "crate" de forma intercambiável com o conceito geral de programação de "biblioteca".

A _raiz do crate_ (_crate root_) é o arquivo-fonte a partir do qual o compilador Rust começa e que forma o módulo raiz do seu crate (vamos explicar módulos em profundidade em [Controlar escopo e privacidade com módulos](/livro/cap07-02-definindo-modulos-para-controlar-escopo-e-privacidade)).

Um _package_ é um pacote de um ou mais crates que fornece um conjunto de funcionalidades. Um package contém um arquivo _Cargo.toml_ que descreve como compilar esses crates. O Cargo, na verdade, também é um package: ele contém o binary crate da ferramenta de linha de comando que você vem usando para compilar seu código. O package do Cargo também inclui um library crate do qual esse binary crate depende. Outros projetos podem depender do library crate do Cargo para reutilizar a mesma lógica usada pela ferramenta de linha de comando do Cargo.

Um package pode conter quantos binary crates você quiser, mas no máximo um único library crate. Um package precisa conter pelo menos um crate, seja library crate ou binary crate.

Vamos ver o que acontece quando criamos um package. Primeiro, executamos o comando `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Depois de executar `cargo new my-project`, usamos `ls` para ver o que o Cargo cria. No diretório _my-project_, existe um arquivo _Cargo.toml_, que define um package. Também existe um diretório _src_ que contém _main.rs_. Abra _Cargo.toml_ no seu editor de texto e perceba que não há nenhuma menção a _src/main.rs_. O Cargo segue a convenção de que _src/main.rs_ é a raiz do crate de um binary crate com o mesmo nome do package. Da mesma forma, o Cargo sabe que, se o diretório do package contiver _src/lib.rs_, então o package contém um library crate com o mesmo nome do package, e _src/lib.rs_ é sua raiz de crate. O Cargo passa os arquivos de raiz de crate para o `rustc` compilar a biblioteca ou o binário.

Aqui, temos um package que contém apenas _src/main.rs_, o que significa que ele tem somente um binary crate chamado `my-project`. Se um package tiver _src/main.rs_ e _src/lib.rs_, ele terá dois crates: um binário e uma biblioteca, ambos com o mesmo nome do package. Um package pode ter vários binary crates ao colocar arquivos no diretório _src/bin_: cada arquivo será um binary crate separado.
