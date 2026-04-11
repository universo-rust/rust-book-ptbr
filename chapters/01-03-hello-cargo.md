---
title: "Hello, Cargo!"
chapter_code: 01-03
slug: hello-cargo
---

# Hello, Cargo!

Cargo é o sistema de build e o gerenciador de pacotes do Rust. A maioria dos Rustaceans usa essa ferramenta para gerenciar seus projetos em Rust porque o Cargo cuida de muitas tarefas para você, como compilar seu código, baixar as bibliotecas das quais seu código depende e compilar essas bibliotecas. (Chamamos as bibliotecas de que seu código precisa de *dependências*.)

Os programas mais simples em Rust, como o que escrevemos até agora, não têm dependências. Se tivéssemos criado o projeto "Hello, world!" com o Cargo, ele usaria apenas a parte do Cargo responsável por compilar o código. À medida que você escreve programas mais complexos em Rust, você adicionará dependências, e se iniciar um projeto usando o Cargo, adicionar dependências será muito mais fácil.

Como a grande maioria dos projetos em Rust usa o Cargo, o restante deste livro assume que você também está usando o Cargo. O Cargo já vem instalado com o Rust se você utilizou os instaladores oficiais discutidos na seção "Instalação". Se você instalou o Rust por outros meios, verifique se o Cargo está instalado digitando o seguinte no terminal:

```bash
$ cargo --version
```

Se você vir um número de versão, ótimo! Se aparecer um erro, como `command not found`, consulte a documentação do método de instalação que você utilizou para descobrir como instalar o Cargo separadamente.

### Criando um projeto com Cargo

Vamos criar um novo projeto usando o Cargo e ver como ele difere do nosso projeto original "Hello, world!". Volte ao diretório onde você costuma armazenar seus projetos e execute os comandos abaixo:

```bash
$ cargo new hello_cargo
$ cd hello_cargo
```

O primeiro comando cria um novo diretório e projeto chamado *hello_cargo*. Demos esse nome ao projeto, e o Cargo cria seus arquivos em um diretório com o mesmo nome.

Entre no diretório *hello_cargo* e liste os arquivos. Você verá que o Cargo gerou dois arquivos e um diretório: um arquivo *Cargo.toml* e um diretório *src* contendo um arquivo *main.rs*.

Ele também inicializou um novo repositório Git junto com um arquivo *.gitignore*. Arquivos do Git não serão gerados se você executar `cargo new` dentro de um repositório Git existente; você pode sobrescrever esse comportamento usando `cargo new --vcs=git`.

> **Nota:** Git é um sistema de controle de versão comum. Você pode configurar o `cargo new` para usar um sistema de controle de versão diferente ou nenhum, utilizando a flag `--vcs`. Execute `cargo new --help` para ver as opções disponíveis.

Abra o arquivo *Cargo.toml* no editor de texto de sua preferência. Ele deve se parecer com o exemplo abaixo:

Arquivo: Cargo.toml
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"
 
[dependencies]
```
[Listagem 1-2](#): Conteúdo do arquivo *Cargo.toml* gerado pelo comando `cargo new`

Esse arquivo utiliza o formato [*TOML*](https://toml.io/) (Tom's Obvious, Minimal Language), que é o formato de configuração do Cargo.

A primeira linha, `[package]`, indica que as declarações seguintes estão configurando um pacote. À medida que adicionarmos mais informações a esse arquivo, novas seções serão incluídas.

As três linhas seguintes definem as informações necessárias para o Cargo compilar seu programa: o nome, a versão e a edition do Rust utilizada. Falaremos mais sobre a chave `edition` no [Apêndice E](#).

A seção `[dependencies]` é onde você lista as dependências do projeto. Em Rust, pacotes de código são chamados de *crates*. Não precisaremos de dependências neste projeto, mas usaremos essa seção no primeiro projeto do Capítulo 2.

Agora abra o arquivo *src/main.rs*:

Arquivo: src/main.rs
```rust
fn main() {
    println!("Hello, world!");
}
```

O Cargo gerou automaticamente um programa "Hello, world!" para você. Até agora, as principais diferenças em relação ao projeto criado manualmente são a organização dos arquivos no diretório *src* e o uso do arquivo de configuração *Cargo.toml*.

O Cargo espera que todo o código-fonte fique dentro do diretório *src*. O diretório principal do projeto é reservado para arquivos como README, licenças e configurações. Isso ajuda a manter o projeto organizado.

Se você começou um projeto sem Cargo, pode convertê-lo facilmente. Basta mover o código para o diretório *src* e criar um arquivo *Cargo.toml*. Uma forma simples de fazer isso é executar `cargo init`.

## Compilando e executando um projeto com Cargo

Agora vamos ver o que muda quando compilamos e executamos o programa **"Hello, world!"** usando o Cargo! A partir do diretório *hello_cargo*, compile seu projeto digitando o seguinte comando:

```bash
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Esse comando cria um arquivo executável em *target/debug/hello_cargo* (ou *target\debug\hello_cargo.exe* no Windows), em vez de criá-lo no diretório atual. Como a compilação padrão é uma compilação de *debug*, o Cargo coloca o binário em um diretório chamado *debug*. Você pode executar o arquivo executável com o seguinte comando:

```bash
$ ./target/debug/hello_cargo # ou .\target\debug\hello_cargo.exe no Windows
Hello, world!
```

Se tudo correr bem, `Hello, world!` deverá ser exibido no terminal. Executar `cargo build` pela primeira vez também faz com que o Cargo crie um novo arquivo no diretório raiz: *Cargo.lock*. Esse arquivo acompanha as versões exatas das dependências do seu projeto. Como este projeto não tem dependências, o arquivo é bem simples. Você nunca precisará alterar esse arquivo manualmente; o Cargo gerencia seu conteúdo para você.

Acabamos de compilar um projeto com `cargo build` e executá-lo com `./target/debug/hello_cargo`, mas também podemos usar `cargo run` para compilar o código e depois executar o binário resultante, tudo em um único comando:

```bash
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Usar `cargo run` é mais conveniente do que ter que lembrar de executar `cargo build` e depois usar o caminho completo até o binário, por isso a maioria dos desenvolvedores utiliza `cargo run`.

Observe que desta vez não vimos nenhuma saída indicando que o Cargo estava compilando o *hello_cargo*. O Cargo percebeu que os arquivos não haviam mudado, então não recompilou o projeto e apenas executou o binário. Se você tivesse modificado o código-fonte, o Cargo teria recompilado o projeto antes de executá-lo, e você teria visto uma saída como esta:

```bash
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

O Cargo também fornece um comando chamado `cargo check`. Esse comando verifica rapidamente seu código para garantir que ele compila, mas não produz um executável:

```bash
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Por que você não iria querer um executável? Muitas vezes, `cargo check` é muito mais rápido que `cargo build`, porque ele pula a etapa de geração do executável. Se você estiver verificando constantemente seu trabalho enquanto escreve o código, usar `cargo check` acelera o processo de saber se o projeto ainda está compilando. Por isso, muitos *Rustaceans* executam `cargo check` periodicamente enquanto escrevem seus programas. Depois, quando estão prontos para usar o executável, executam `cargo build`.

Vamos recapitular o que aprendemos até agora sobre o Cargo:

- Podemos criar um projeto usando `cargo new`.
- Podemos compilar um projeto usando `cargo build`.
- Podemos compilar e executar um projeto em um único passo usando `cargo run`.
- Podemos verificar se um projeto compila sem gerar um binário usando `cargo check`.
- Em vez de salvar o resultado da compilação no mesmo diretório do código, o Cargo o armazena no diretório *target/debug*.

Uma vantagem adicional de usar o Cargo é que os comandos são os mesmos, independentemente do sistema operacional. Por isso, a partir daqui, não forneceremos mais instruções específicas para Linux e macOS versus Windows.

### Compilando para release

Quando seu projeto estiver finalmente pronto para lançamento, você pode usar `cargo build --release` para compilá-lo com otimizações. Esse comando criará um executável em *target/release* em vez de *target/debug*. As otimizações fazem com que o código Rust rode mais rápido, mas aumentam o tempo de compilação. Por isso existem dois perfis diferentes: um para desenvolvimento, quando você quer recompilar rapidamente e com frequência, e outro para gerar o programa final que será entregue ao usuário. Se você estiver medindo o tempo de execução do código, certifique-se de executar `cargo build --release` e medir usando o executável em *target/release*.

### Aproveitando as convenções do Cargo

Em projetos simples, o Cargo não oferece muito mais valor do que usar apenas o `rustc`, mas ele mostra seu verdadeiro valor conforme os programas se tornam mais complexos. Quando os projetos crescem para múltiplos arquivos ou precisam de dependências, é muito mais fácil deixar o Cargo coordenar a compilação.

Mesmo que o projeto *hello_cargo* seja simples, ele já utiliza grande parte das ferramentas reais que você usará ao longo da sua carreira com Rust. Para trabalhar em qualquer projeto existente, você pode usar os seguintes comandos:

```bash
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Para mais informações sobre o Cargo, consulte [a documentação oficial](https://doc.rust-lang.org/cargo/).

## Resumo

Você já começou muito bem sua jornada com Rust! Neste capítulo, você aprendeu a:

- Instalar a versão estável mais recente do Rust usando `rustup`.
- Atualizar para uma versão mais nova do Rust.
- Abrir a documentação instalada localmente.
- Escrever e executar um programa **"Hello, world!"** usando o `rustc` diretamente.
- Criar e executar um novo projeto usando as convenções do Cargo.

Este é um ótimo momento para criar um programa mais robusto e se acostumar a ler e escrever código Rust. Portanto, no Capítulo 2, construiremos um jogo de adivinhação. Se preferir começar aprendendo como os conceitos comuns de programação funcionam em Rust, consulte o Capítulo 3 e depois retorne ao Capítulo 2.