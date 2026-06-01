---
title: "Trazendo caminhos para o escopo com `use`"
chapter_code: 07-04
slug: trazendo-caminhos-para-o-escopo-com-use
---

# Trazendo Caminhos para o Escopo com a Palavra-chave `use`

Ter que escrever caminhos completos para chamar funções pode ser incômodo e repetitivo. Na Listagem 7-7, independentemente de termos escolhido o caminho absoluto ou relativo para a função `add_to_waitlist`, toda vez que quiséssemos chamá-la precisaríamos especificar `front_of_house` e `hosting` também. Felizmente, existe uma forma de simplificar isso: podemos criar uma vez um atalho para um caminho com a palavra-chave `use` e, depois, usar o nome mais curto no restante do escopo.

Na Listagem 7-11, trazemos o módulo `crate::front_of_house::hosting` para o escopo em que `eat_at_restaurant` está definida, para que só precisemos escrever `hosting::add_to_waitlist` ao chamar a função `add_to_waitlist` em `eat_at_restaurant`.

**Arquivo: src/lib.rs**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

<a id="listagem-7-11"></a>

[Listagem 7-11](#listagem-7-11): Trazendo um módulo para o escopo com `use`

Adicionar `use` com um caminho em um escopo é semelhante a criar um link simbólico no sistema de arquivos. Ao adicionar `use crate::front_of_house::hosting` na raiz do crate, `hosting` passa a ser um nome válido nesse escopo, como se o módulo `hosting` tivesse sido definido na raiz do crate. Caminhos trazidos para o escopo com `use` também obedecem às regras de privacidade, como qualquer outro caminho.

Observe que `use` cria o atalho apenas para o escopo específico em que a declaração aparece. A Listagem 7-12 move a função `eat_at_restaurant` para um novo módulo filho chamado `customer`, que está em um escopo diferente da declaração `use`; por isso, o corpo da função não compila.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

<a id="listagem-7-12"></a>

[Listagem 7-12](#listagem-7-12): Uma declaração `use` só se aplica no escopo em que está

O erro do compilador mostra que o atalho não se aplica mais dentro do módulo `customer`:

```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0433]: failed to resolve: use of unresolved module or unlinked crate `hosting`
  --> src/lib.rs:11:9
   |
11 |         hosting::add_to_waitlist();
   |         ^^^^^^^ use of unresolved module or unlinked crate `hosting`
   |
   = help: if you wanted to use a crate named `hosting`, use `cargo add hosting` to add it to your `Cargo.toml`
help: consider importing this module through its public re-export
   |
10 +     use crate::hosting;
   |

warning: unused import: `crate::front_of_house::hosting`
 --> src/lib.rs:7:5
  |
7 | use crate::front_of_house::hosting;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

For more information about this error, try `rustc --explain E0433`.
warning: `restaurant` (lib) generated 1 warning
error: could not compile `restaurant` (lib) due to 1 previous error; 1 warning emitted
```

Observe que também há um aviso de que o `use` não está mais sendo usado no escopo em que foi declarado! Para corrigir esse problema, mova o `use` para dentro do módulo `customer` também, ou referencie o atalho do módulo pai com `super::hosting` dentro do módulo filho `customer`.

### Criando caminhos `use` idiomáticos

Na Listagem 7-11, você pode ter se perguntado por que especificamos `use crate::front_of_house::hosting` e depois chamamos `hosting::add_to_waitlist` em `eat_at_restaurant`, em vez de especificar no `use` o caminho completo até a função `add_to_waitlist`, para obter o mesmo resultado, como na Listagem 7-13.

**Arquivo: src/lib.rs**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

<a id="listagem-7-13"></a>

[Listagem 7-13](#listagem-7-13): Trazendo a função `add_to_waitlist` para o escopo com `use`, o que não é idiomático

Embora a Listagem 7-11 e a Listagem 7-13 realizem a mesma tarefa, a Listagem 7-11 é a forma idiomática de trazer uma função para o escopo com `use`. Trazer para o escopo o módulo pai da função significa que precisamos especificar esse módulo ao chamar a função. Isso deixa claro que a função não está definida localmente e, ao mesmo tempo, minimiza a repetição do caminho completo. No código da Listagem 7-13, fica menos claro onde `add_to_waitlist` está definida.

Por outro lado, ao trazer structs, enums e outros itens com `use`, o idiomático é especificar o caminho completo. A Listagem 7-14 mostra a forma idiomática de trazer a struct `HashMap` da biblioteca padrão para o escopo de um binary crate.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

<a id="listagem-7-14"></a>

[Listagem 7-14](#listagem-7-14): Trazendo `HashMap` para o escopo de forma idiomática

Não há um motivo forte por trás desse estilo: é apenas a convenção que surgiu, e as pessoas se acostumaram a ler e escrever código Rust dessa forma.

A exceção a esse estilo é quando estamos trazendo para o escopo, com declarações `use`, dois itens com o mesmo nome, porque o Rust não permite isso. A Listagem 7-15 mostra como trazer dois tipos `Result` com o mesmo nome, mas com módulos pais diferentes, e como referenciá-los.

**Arquivo: src/lib.rs**

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    Ok(())
}

fn function2() -> io::Result<()> {
    Ok(())
}
```

<a id="listagem-7-15"></a>

[Listagem 7-15](#listagem-7-15): Trazer dois tipos com o mesmo nome para o mesmo escopo exige usar seus módulos pais

Como você pode ver, usar os módulos pais diferencia os dois tipos `Result`. Se, em vez disso, especificássemos `use std::fmt::Result` e `use std::io::Result`, teríamos dois tipos `Result` no mesmo escopo, e o Rust não saberia a qual deles estamos nos referindo ao usar `Result`.

### Fornecendo novos nomes com a palavra-chave `as`

Há outra solução para o problema de trazer, com `use`, dois tipos com o mesmo nome para o mesmo escopo: depois do caminho, podemos especificar `as` e um novo nome local, ou _alias_, para o tipo. A Listagem 7-16 mostra outra forma de escrever o código da Listagem 7-15, renomeando um dos dois tipos `Result` com `as`.

**Arquivo: src/lib.rs**

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    Ok(())
}

fn function2() -> IoResult<()> {
    Ok(())
}
```

<a id="listagem-7-16"></a>

[Listagem 7-16](#listagem-7-16): Renomeando um tipo quando ele é trazido para o escopo com a palavra-chave `as`

Na segunda declaração `use`, escolhemos o novo nome `IoResult` para o tipo `std::io::Result`, que não entra em conflito com o `Result` de `std::fmt`, também trazido para o escopo. Tanto a Listagem 7-15 quanto a Listagem 7-16 são consideradas idiomáticas, então a escolha é sua!

### Reexportando nomes com `pub use`

Quando trazemos um nome para o escopo com a palavra-chave `use`, esse nome fica privado ao escopo para o qual foi importado. Para permitir que código fora desse escopo se refira a esse nome como se ele tivesse sido definido ali, podemos combinar `pub` e `use`. Essa técnica é chamada de _reexportação_, porque estamos trazendo um item para o escopo e também tornando esse item disponível para outros o trazerem para seus próprios escopos.

A Listagem 7-17 mostra o código da Listagem 7-11 com `use` no módulo raiz alterado para `pub use`.

**Arquivo: src/lib.rs**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

<a id="listagem-7-17"></a>

[Listagem 7-17](#listagem-7-17): Tornando um nome disponível para qualquer código usar a partir de um novo escopo com `pub use`

Antes dessa mudança, código externo teria que chamar a função `add_to_waitlist` usando o caminho `restaurant::front_of_house::hosting::add_to_waitlist()`, o que também exigiria que o módulo `front_of_house` fosse marcado como `pub`. Agora que esse `pub use` reexportou o módulo `hosting` a partir do módulo raiz, código externo pode usar o caminho `restaurant::hosting::add_to_waitlist()`.

Reexportar é útil quando a estrutura interna do seu código é diferente da forma como quem consome seu código pensa sobre o domínio. Por exemplo, nesta metáfora de restaurante, quem trabalha no restaurante pensa em “front of house” e “back of house”. Já clientes que visitam o restaurante provavelmente não pensam nessas divisões nesses termos. Com `pub use`, podemos escrever o código com uma estrutura e expor outra. Isso deixa a biblioteca bem organizada tanto para quem trabalha nela quanto para quem a usa. Veremos outro exemplo de `pub use` e como ele afeta a documentação do crate em "Exportando uma API pública conveniente", no Capítulo 14.

### Usando packages externos

No Capítulo 2, programamos um projeto de jogo de adivinhação que usou um package externo chamado `rand` para obter números aleatórios. Para usar `rand` no projeto, adicionamos esta linha a _Cargo.toml_:

**Arquivo: Cargo.toml**

```toml
[dependencies]
rand = "0.8.5"
```

Adicionar `rand` como dependência em _Cargo.toml_ diz ao Cargo para baixar o package `rand`, bem como suas dependências, de [crates.io](https://crates.io/), e tornar `rand` disponível para o projeto.

Em seguida, para trazer definições de `rand` para o escopo do nosso package, adicionamos uma linha `use` começando com o nome do crate, `rand`, e listamos os itens que queríamos importar. Lembre-se de que, na seção "Gerando um número aleatório", do Capítulo 2, trouxemos a trait `Rng` para o escopo e chamamos a função `rand::thread_rng`:

```rust
use rand::Rng;

// --snip--

let secret_number = rand::thread_rng().gen_range(1..=100);
```

A comunidade Rust disponibilizou muitos packages em [crates.io](https://crates.io/), e trazer qualquer um deles para o seu package envolve os mesmos passos: listá-los no _Cargo.toml_ do package e usar `use` para trazer itens desses crates para o escopo.

Observe que a biblioteca padrão `std` também é um crate externo ao nosso package. Como a biblioteca padrão já vem com a linguagem Rust, não precisamos alterar _Cargo.toml_ para incluir `std`. Mas precisamos referenciá-la com `use` para trazer itens dela para o escopo do nosso package. Por exemplo, com `HashMap`, usaríamos esta linha:

```rust
use std::collections::HashMap;
```

Este é um caminho absoluto começando com `std`, o nome do crate da biblioteca padrão.

### Usando caminhos aninhados para limpar listas `use`

Se estivermos usando vários itens definidos no mesmo crate ou no mesmo módulo, listar cada item em sua própria linha pode ocupar muito espaço vertical no arquivo. Por exemplo, estas duas declarações `use` que tínhamos no jogo de adivinhação, na Listagem 2-4, trazem itens de `std` para o escopo:

**Arquivo: src/main.rs**

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;
```

Em vez disso, podemos usar caminhos aninhados para trazer os mesmos itens para o escopo em uma única linha. Fazemos isso especificando a parte comum do caminho, seguida por dois-pontos duplos, e depois chaves envolvendo uma lista com as partes que diferem, como mostrado na Listagem 7-18.

**Arquivo: src/main.rs**

```rust
use rand::Rng;
use std::{cmp::Ordering, io};
```

<a id="listagem-7-18"></a>

[Listagem 7-18](#listagem-7-18): Especificando um caminho aninhado para trazer vários itens com o mesmo prefixo para o escopo

Em programas maiores, trazer muitos itens para o escopo de um mesmo crate ou módulo usando caminhos aninhados pode reduzir bastante o número de declarações `use` separadas necessárias!

Podemos usar um caminho aninhado em qualquer nível do caminho, o que é útil ao combinar duas declarações `use` que compartilham um subcaminho. Por exemplo, a Listagem 7-19 mostra duas declarações `use`: uma que traz `std::io` para o escopo e outra que traz `std::io::Write`.

**Arquivo: src/lib.rs**

```rust
use std::io;
use std::io::Write;
```

<a id="listagem-7-19"></a>

[Listagem 7-19](#listagem-7-19): Duas declarações `use` em que uma é um subcaminho da outra

A parte comum desses dois caminhos é `std::io`, e esse é o primeiro caminho completo. Para mesclar esses dois caminhos em uma declaração `use`, podemos usar `self` no caminho aninhado, como mostrado na Listagem 7-20.

**Arquivo: src/lib.rs**

```rust
use std::io::{self, Write};
```

<a id="listagem-7-20"></a>

[Listagem 7-20](#listagem-7-20): Combinando os caminhos da Listagem 7-19 em uma declaração `use`

Esta linha traz `std::io` e `std::io::Write` para o escopo.

### Importando itens com o operador glob

Se quisermos trazer _todos_ os itens públicos definidos em um caminho para o escopo, podemos especificar esse caminho seguido pelo operador glob `*`:

```rust
use std::collections::*;
```

Essa declaração `use` traz todos os itens públicos definidos em `std::collections` para o escopo atual. Tenha cuidado ao usar o operador glob! Ele pode dificultar saber quais nomes estão no escopo e onde um nome usado no programa foi definido. Além disso, se a dependência mudar suas definições, o que você importou também muda, e isso pode gerar erros de compilação ao atualizar a dependência, por exemplo, se ela adicionar uma definição com o mesmo nome de uma definição sua no mesmo escopo.

O operador glob é frequentemente usado em testes para trazer tudo que está sob teste para o módulo `tests`; falaremos sobre isso em "Como escrever testes", no Capítulo 11. O operador glob também é às vezes usado como parte do padrão prelude: consulte a documentação da biblioteca padrão para mais informações sobre esse padrão.
