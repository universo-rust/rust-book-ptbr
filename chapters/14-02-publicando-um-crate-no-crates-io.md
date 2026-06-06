---
title: "Publicando um crate no Crates.io"
chapter_code: 14-02
slug: publicando-um-crate-no-crates-io
challenge_day: 18
reading_minutes: 20
---

# Publicando um crate no Crates.io

Usamos pacotes de [crates.io](https://crates.io/) como dependências do nosso projeto, mas você também pode compartilhar seu código com outras pessoas publicando seus próprios pacotes. O registro de crates em [crates.io](https://crates.io/) distribui o código-fonte dos seus pacotes, então ele hospeda principalmente código open source.

O Rust e o Cargo têm recursos que tornam seu pacote publicado mais fácil de encontrar e usar. Falaremos de alguns desses recursos a seguir e depois explicaremos como publicar um pacote.

## Escrevendo comentários de documentação úteis

Documentar seus pacotes com precisão ajuda outros usuários a saber como e quando usá-los, então vale a pena investir tempo escrevendo documentação. No Capítulo 3, discutimos como comentar código Rust usando duas barras, `//`. O Rust também tem um tipo particular de comentário para documentação, convenientemente chamado de _comentário de documentação_, que gera documentação HTML. O HTML exibe o conteúdo dos comentários de documentação para itens de API pública destinados a programadores interessados em saber como _usar_ seu crate, em oposição a como seu crate é _implementado_.

Comentários de documentação usam três barras, `///`, em vez de duas e suportam notação Markdown para formatar o texto. Coloque comentários de documentação logo antes do item que estão documentando. A Listagem 14-1 mostra comentários de documentação para uma função `add_one` em um crate chamado `my_crate`.

**Arquivo: src/lib.rs**

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

<a id="listagem-14-1"></a>

[Listagem 14-1](#listagem-14-1): Um comentário de documentação para uma função

Aqui, damos uma descrição do que a função `add_one` faz, iniciamos uma seção com o título `Examples` e então fornecemos código que demonstra como usar a função `add_one`. Podemos gerar a documentação HTML a partir deste comentário de documentação executando `cargo doc`. Este comando executa a ferramenta `rustdoc` distribuída com o Rust e coloca a documentação HTML gerada no diretório _target/doc_.

Por conveniência, executar `cargo doc --open` compilará o HTML da documentação do seu crate atual (bem como a documentação de todas as dependências do seu crate) e abrirá o resultado em um navegador web. Navegue até a função `add_one` e verá como o texto nos comentários de documentação é renderizado, como mostrado na Figura 14-1.

![Documentação HTML renderizada para a função `add_one` do crate `my_crate`](https://doc.rust-lang.org/book/img/trpl14-01.png)

*Figura 14-1: A documentação HTML da função `add_one`*

### Seções comumente usadas

Usamos o título Markdown `# Examples` na Listagem 14-1 para criar uma seção no HTML com o título “Examples”. Aqui estão outras seções que autores de crates costumam usar em sua documentação:

- **Panics**: Cenários em que a função documentada pode entrar em pânico. Quem chama a função e não quer que o programa entre em pânico deve garantir que não chame a função nessas situações.
- **Errors**: Se a função retorna um `Result`, descrever os tipos de erros que podem ocorrer e quais condições podem causar o retorno desses erros pode ajudar quem chama a função a escrever código que trate os diferentes tipos de erro de formas diferentes.
- **Safety**: Se a função é `unsafe` para chamar (discutimos _unsafety_ no Capítulo 20), deve haver uma seção explicando por que a função é unsafe e cobrindo os invariantes que a função espera que quem chama mantenha.

A maioria dos comentários de documentação não precisa de todas essas seções, mas esta é uma boa lista de verificação para lembrar os aspectos do seu código que os usuários terão interesse em conhecer.

### Comentários de documentação como testes

Adicionar blocos de código de exemplo nos seus comentários de documentação pode ajudar a demonstrar como usar sua biblioteca e tem um bônus adicional: executar `cargo test` executará os exemplos de código na sua documentação como testes! Nada é melhor do que documentação com exemplos. Mas nada é pior do que exemplos que não funcionam porque o código mudou desde que a documentação foi escrita. Se executarmos `cargo test` com a documentação da função `add_one` da Listagem 14-1, veremos uma seção nos resultados dos testes parecida com isto:

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Agora, se mudarmos a função ou o exemplo para que o `assert_eq!` no exemplo entre em pânico, e executarmos `cargo test` novamente, veremos que os doc tests detectam que o exemplo e o código estão fora de sincronia!

### Comentários de itens contidos

O estilo de comentário de documentação `//!` adiciona documentação ao item que _contém_ os comentários, em vez dos itens _seguintes_ aos comentários. Normalmente usamos esses comentários de documentação dentro do arquivo raiz do crate (_src/lib.rs_ por convenção) ou dentro de um módulo para documentar o crate ou o módulo como um todo.

Por exemplo, para adicionar documentação que descreve o propósito do crate `my_crate` que contém a função `add_one`, adicionamos comentários de documentação que começam com `//!` no início do arquivo _src/lib.rs_, como mostrado na Listagem 14-2.

**Arquivo: src/lib.rs**

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

<a id="listagem-14-2"></a>

[Listagem 14-2](#listagem-14-2): A documentação do crate `my_crate` como um todo

Observe que não há código após a última linha que começa com `//!`. Como começamos os comentários com `//!` em vez de `///`, estamos documentando o item que contém este comentário, em vez de um item que segue este comentário. Neste caso, esse item é o arquivo _src/lib.rs_, que é a raiz do crate. Esses comentários descrevem o crate inteiro.

Quando executarmos `cargo doc --open`, esses comentários serão exibidos na página inicial da documentação de `my_crate`, acima da lista de itens públicos do crate, como mostrado na Figura 14-2.

Comentários de documentação dentro de itens são úteis especialmente para descrever crates e módulos. Use-os para explicar o propósito geral do contêiner e ajudar seus usuários a entender a organização do crate.

![Documentação HTML renderizada com um comentário para o crate como um todo](https://doc.rust-lang.org/book/img/trpl14-02.png)

*Figura 14-2: A documentação renderizada de `my_crate`, incluindo o comentário que descreve o crate como um todo*

## Exportando uma API pública conveniente

A estrutura da sua API pública é uma consideração importante ao publicar um crate. Pessoas que usam seu crate estão menos familiarizadas com a estrutura do que você e podem ter dificuldade em encontrar as partes que querem usar se seu crate tiver uma hierarquia de módulos grande.

No Capítulo 7, cobrimos como tornar itens públicos usando a palavra-chave `pub` e como trazer itens para um escopo com a palavra-chave `use`. No entanto, a estrutura que faz sentido para você enquanto desenvolve um crate pode não ser muito conveniente para seus usuários. Você pode querer organizar suas structs em uma hierarquia com vários níveis, mas então pessoas que querem usar um tipo que você definiu profundamente na hierarquia podem ter dificuldade em descobrir que o tipo existe. Elas também podem se irritar em ter que digitar `use my_crate::some_module::another_module::UsefulType;` em vez de `use my_crate::UsefulType;`.

A boa notícia é que, se a estrutura _não_ for conveniente para outras pessoas usarem a partir de outra biblioteca, você não precisa reorganizar sua organização interna: em vez disso, pode reexportar itens para criar uma estrutura pública diferente da sua estrutura privada usando `pub use`. _Reexportar_ pega um item público em um local e o torna público em outro local, como se tivesse sido definido nesse outro local.

Por exemplo, digamos que fizemos uma biblioteca chamada `art` para modelar conceitos artísticos. Dentro desta biblioteca há dois módulos: um módulo `kinds` contendo dois enums chamados `PrimaryColor` e `SecondaryColor` e um módulo `utils` contendo uma função chamada `mix`, como mostrado na Listagem 14-3.

**Arquivo: src/lib.rs**

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        unimplemented!();
    }
}
```

<a id="listagem-14-3"></a>

[Listagem 14-3](#listagem-14-3): Uma biblioteca `art` com itens organizados nos módulos `kinds` e `utils`

A Figura 14-3 mostra como seria a página inicial da documentação deste crate gerada por `cargo doc`.

![Documentação renderizada do crate `art` que lista os módulos `kinds` e `utils`](https://doc.rust-lang.org/book/img/trpl14-03.png)

*Figura 14-3: A página inicial da documentação de `art` que lista os módulos `kinds` e `utils`*

Note que os tipos `PrimaryColor` e `SecondaryColor` não estão listados na página inicial, nem a função `mix`. Precisamos clicar em `kinds` e `utils` para vê-los.

Outro crate que depende desta biblioteca precisaria de instruções `use` que tragam os itens de `art` para o escopo, especificando a estrutura de módulos atualmente definida. A Listagem 14-4 mostra um exemplo de crate que usa os itens `PrimaryColor` e `mix` do crate `art`.

**Arquivo: src/main.rs**

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

<a id="listagem-14-4"></a>

[Listagem 14-4](#listagem-14-4): Um crate usando os itens do crate `art` com sua estrutura interna exportada

O autor do código na Listagem 14-4, que usa o crate `art`, teve que descobrir que `PrimaryColor` está no módulo `kinds` e `mix` está no módulo `utils`. A estrutura de módulos do crate `art` é mais relevante para desenvolvedores trabalhando no crate `art` do que para quem o usa. A estrutura interna não contém informação útil para alguém tentando entender como usar o crate `art`; em vez disso, causa confusão porque quem o usa precisa descobrir onde procurar e deve especificar os nomes dos módulos nas instruções `use`.

Para remover a organização interna da API pública, podemos modificar o código do crate `art` na Listagem 14-3 para adicionar instruções `pub use` que reexportam os itens no nível superior, como mostrado na Listagem 14-5.

**Arquivo: src/lib.rs**

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        SecondaryColor::Orange
    }
}
```

<a id="listagem-14-5"></a>

[Listagem 14-5](#listagem-14-5): Adicionando instruções `pub use` para reexportar itens

A documentação da API que `cargo doc` gera para este crate agora listará e vinculará reexportações na página inicial, como mostrado na Figura 14-4, tornando os tipos `PrimaryColor` e `SecondaryColor` e a função `mix` mais fáceis de encontrar.

![Documentação renderizada do crate `art` com as reexportações na página inicial](https://doc.rust-lang.org/book/img/trpl14-04.png)

*Figura 14-4: A página inicial da documentação de `art` que lista as reexportações*

Usuários do crate `art` ainda podem ver e usar a estrutura interna da Listagem 14-3, como demonstrado na Listagem 14-4, ou podem usar a estrutura mais conveniente da Listagem 14-5, como mostrado na Listagem 14-6.

**Arquivo: src/main.rs**

```rust
use art::PrimaryColor;
use art::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

<a id="listagem-14-6"></a>

[Listagem 14-6](#listagem-14-6): Um programa usando os itens reexportados do crate `art`

Em casos em que há muitos módulos aninhados, reexportar os tipos no nível superior com `pub use` pode fazer uma diferença significativa na experiência de quem usa o crate. Outro uso comum de `pub use` é reexportar definições de uma dependência no crate atual para tornar as definições daquele crate parte da API pública do seu crate.

Criar uma estrutura de API pública útil é mais arte do que ciência, e você pode iterar para encontrar a API que funciona melhor para seus usuários. Escolher `pub use` dá flexibilidade em como estrutura seu crate internamente e desacopla essa estrutura interna do que você apresenta aos seus usuários. Olhe o código de alguns crates que você instalou para ver se a estrutura interna deles difere da API pública.

## Configurando uma conta no Crates.io

Antes de publicar qualquer crate, você precisa criar uma conta em [crates.io](https://crates.io/) e obter um token de API. Para isso, visite a página inicial em [crates.io](https://crates.io/) e faça login via conta GitHub. (A conta GitHub é atualmente um requisito, mas o site pode suportar outras formas de criar conta no futuro.) Depois de logado, visite as configurações da sua conta em [https://crates.io/me/](https://crates.io/me/) e recupere sua chave de API. Em seguida, execute o comando `cargo login` e cole sua chave de API quando solicitado, assim:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Este comando informará o Cargo do seu token de API e o armazenará localmente em _~/.cargo/credentials.toml_. Note que este token é um segredo: não o compartilhe com ninguém. Se compartilhar com alguém por qualquer motivo, você deve revogá-lo e gerar um novo token em [crates.io](https://crates.io/).

## Adicionando metadados a um novo crate

Digamos que você tem um crate que quer publicar. Antes de publicar, precisará adicionar alguns metadados na seção `[package]` do arquivo _Cargo.toml_ do crate.

Seu crate precisará de um nome único. Enquanto trabalha em um crate localmente, você pode nomear um crate como quiser. No entanto, nomes de crates em [crates.io](https://crates.io/) são alocados por ordem de chegada. Depois que um nome de crate é tomado, ninguém mais pode publicar um crate com esse nome. Antes de tentar publicar um crate, pesquise o nome que quer usar. Se o nome já foi usado, precisará encontrar outro nome e editar o campo `name` no _Cargo.toml_ na seção `[package]` para usar o novo nome para publicação, assim:

**Arquivo: Cargo.toml**

```toml
[package]
name = "guessing_game"
```

Mesmo que tenha escolhido um nome único, quando executar `cargo publish` para publicar o crate neste ponto, receberá um aviso e depois um erro:

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Isso resulta em erro porque faltam algumas informações cruciais: uma descrição e uma licença são exigidas para que as pessoas saibam o que seu crate faz e sob quais termos podem usá-lo. No _Cargo.toml_, adicione uma descrição de apenas uma ou duas frases, porque ela aparecerá com seu crate nos resultados de busca. Para o campo `license`, você precisa fornecer um _valor identificador de licença_. A [Software Package Data Exchange (SPDX) da Linux Foundation](https://spdx.org/licenses/) lista os identificadores que você pode usar para este valor. Por exemplo, para especificar que licenciou seu crate usando a Licença MIT, adicione o identificador `MIT`:

**Arquivo: Cargo.toml**

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Se quiser usar uma licença que não aparece na SPDX, precisa colocar o texto dessa licença em um arquivo, incluir o arquivo no seu projeto e então usar `license-file` para especificar o nome desse arquivo em vez de usar a chave `license`.

Orientação sobre qual licença é apropriada para seu projeto está além do escopo deste livro. Muitas pessoas na comunidade Rust licenciam seus projetos da mesma forma que o Rust, usando uma licença dupla `MIT OR Apache-2.0`. Esta prática demonstra que você também pode especificar vários identificadores de licença separados por `OR` para ter várias licenças para seu projeto.

Com um nome único, a versão, sua descrição e uma licença adicionados, o _Cargo.toml_ de um projeto pronto para publicar pode se parecer com isto:

**Arquivo: Cargo.toml**

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

A [documentação do Cargo](https://doc.rust-lang.org/cargo/) descreve outros metadados que você pode especificar para garantir que outras pessoas descubram e usem seu crate com mais facilidade.

## Publicando no Crates.io

Agora que você criou uma conta, salvou seu token de API, escolheu um nome para seu crate e especificou os metadados exigidos, está pronto para publicar! Publicar um crate envia uma versão específica para [crates.io](https://crates.io/) para que outras pessoas a usem.

Tenha cuidado, porque uma publicação é _permanente_. A versão nunca pode ser sobrescrita, e o código não pode ser excluído exceto em certas circunstâncias. Um dos principais objetivos do Crates.io é atuar como um arquivo permanente de código para que compilações de todos os projetos que dependem de crates de [crates.io](https://crates.io/) continuem funcionando. Permitir exclusão de versões tornaria impossível cumprir esse objetivo. No entanto, não há limite para o número de versões de crate que você pode publicar.

Execute o comando `cargo publish` novamente. Deve ter sucesso agora:

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Parabéns! Você agora compartilhou seu código com a comunidade Rust, e qualquer pessoa pode facilmente adicionar seu crate como dependência de seu projeto.

## Publicando uma nova versão de um crate existente

Quando você fez mudanças no seu crate e está pronto para lançar uma nova versão, altera o valor `version` especificado no seu _Cargo.toml_ e republica. Use as [regras de versionamento semântico](https://semver.org/) para decidir qual é o próximo número de versão apropriado, com base nos tipos de mudanças que fez. Depois, execute `cargo publish` para enviar a nova versão.

## Depreciando versões do Crates.io

Embora você não possa remover versões anteriores de um crate, pode impedir que projetos futuros as adicionem como nova dependência. Isso é útil quando uma versão de crate está quebrada por algum motivo. Nesses casos, o Cargo suporta _yank_ de uma versão de crate.

_Fazer yank_ de uma versão impede que novos projetos dependam dessa versão, permitindo que todos os projetos existentes que dependem dela continuem. Essencialmente, yank significa que todos os projetos com um _Cargo.lock_ não quebrarão, e quaisquer _Cargo.lock_ gerados no futuro não usarão a versão com yank.

Para fazer yank de uma versão de um crate, no diretório do crate que você publicou anteriormente, execute `cargo yank` e especifique qual versão quer fazer yank. Por exemplo, se publicamos um crate chamado `guessing_game` versão 1.0.1 e queremos fazer yank, executaríamos o seguinte no diretório do projeto `guessing_game`:

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Adicionando `--undo` ao comando, você também pode desfazer um yank e permitir que projetos comecem a depender de uma versão novamente:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Um yank _não_ exclui nenhum código. Não pode, por exemplo, excluir segredos enviados acidentalmente. Se isso acontecer, você deve redefinir esses segredos imediatamente.
