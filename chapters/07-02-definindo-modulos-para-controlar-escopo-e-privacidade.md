---
title: "Controlar escopo e privacidade com módulos"
chapter_code: 07-02
slug: definindo-modulos-para-controlar-escopo-e-privacidade
---

# Controlar Escopo e Privacidade com Módulos

Nesta seção, falaremos sobre módulos e outras partes do sistema de módulos, a saber _caminhos_, que permitem nomear itens; a palavra-chave `use`, que traz um caminho para o escopo; e a palavra-chave `pub` para tornar itens públicos. Também discutiremos a palavra-chave `as`, packages externos e o operador glob.

### Folha de consulta de módulos

Antes de entrar nos detalhes de módulos e caminhos, aqui está uma referência rápida de como módulos, caminhos, a palavra-chave `use` e a palavra-chave `pub` funcionam no compilador, e como a maioria dos desenvolvedores organiza seu código. Veremos exemplos de cada uma dessas regras ao longo deste capítulo, mas este é um ótimo lugar para consultar como lembrete de como módulos funcionam.

- **Comece pela raiz do crate**: ao compilar um crate, o compilador primeiro procura no arquivo de raiz do crate (geralmente _src/lib.rs_ para um library crate e _src/main.rs_ para um binary crate) o código a compilar.
- **Declarando módulos**: no arquivo de raiz do crate, você pode declarar novos módulos; digamos que declare um módulo “garden” com `mod garden;`. O compilador procurará o código do módulo nestes lugares:
  - Inline, dentro de chaves que substituem o ponto e vírgula após `mod garden`
  - No arquivo _src/garden.rs_
  - No arquivo _src/garden/mod.rs_
- **Declarando submódulos**: em qualquer arquivo que não seja a raiz do crate, você pode declarar submódulos. Por exemplo, pode declarar `mod vegetables;` em _src/garden.rs_. O compilador procurará o código do submódulo no diretório nomeado para o módulo pai nestes lugares:
  - Inline, diretamente após `mod vegetables`, dentro de chaves em vez do ponto e vírgula
  - No arquivo _src/garden/vegetables.rs_
  - No arquivo _src/garden/vegetables/mod.rs_
- **Caminhos para código em módulos**: uma vez que um módulo faz parte do seu crate, você pode referenciar código nesse módulo de qualquer outro lugar no mesmo crate, desde que as regras de privacidade permitam, usando o caminho para o código. Por exemplo, um tipo `Asparagus` no módulo garden vegetables seria encontrado em `crate::garden::vegetables::Asparagus`.
- **Privado vs. público**: código dentro de um módulo é privado em relação aos seus módulos pais por padrão. Para tornar um módulo público, declare-o com `pub mod` em vez de `mod`. Para tornar itens dentro de um módulo público também públicos, use `pub` antes de suas declarações.
- **A palavra-chave `use`**: dentro de um escopo, a palavra-chave `use` cria atalhos para itens para reduzir a repetição de caminhos longos. Em qualquer escopo que possa referenciar `crate::garden::vegetables::Asparagus`, você pode criar um atalho com `use crate::garden::vegetables::Asparagus;`, e a partir daí só precisa escrever `Asparagus` para usar esse tipo no escopo.

Aqui, criamos um binary crate chamado `backyard` que ilustra essas regras. O diretório do crate, também chamado _backyard_, contém estes arquivos e diretórios:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

O arquivo de raiz do crate neste caso é _src/main.rs_, e contém:

**Arquivo: src/main.rs**

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {plant:?}!");
}
```

A linha `pub mod garden;` diz ao compilador para incluir o código que encontra em _src/garden.rs_, que é:

**Arquivo: src/garden.rs**

```rust
pub mod vegetables;
```

Aqui, `pub mod vegetables;` significa que o código em _src/garden/vegetables.rs_ também é incluído. Esse código é:

**Arquivo: src/garden/vegetables.rs**

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

Agora vamos aos detalhes dessas regras e demonstrá-las em ação!

### Agrupando código relacionado em módulos

_Módulos_ nos permitem organizar código dentro de um crate para legibilidade e reutilização fácil. Módulos também nos permitem controlar a _privacidade_ dos itens porque o código dentro de um módulo é privado por padrão. Itens privados são detalhes de implementação interna não disponíveis para uso externo. Podemos escolher tornar módulos e os itens dentro deles públicos, o que os expõe para permitir que código externo os use e dependa deles.

Como exemplo, vamos escrever um library crate que fornece a funcionalidade de um restaurante. Definiremos as assinaturas de funções, mas deixaremos seus corpos vazios para nos concentrar na organização do código em vez da implementação de um restaurante.

Na indústria de restaurantes, algumas partes de um restaurante são chamadas de _front of house_ (salão) e outras de _back of house_ (cozinha). O _front of house_ é onde os clientes estão; abrange onde os anfitriões acomodam clientes, garçons tomam pedidos e pagamento, e bartenders fazem bebidas. O _back of house_ é onde chefs e cozinheiros trabalham na cozinha, lavadores de louça limpam, e gerentes fazem trabalho administrativo.

Para estruturar nosso crate dessa forma, podemos organizar suas funções em módulos aninhados. Crie uma nova biblioteca chamada `restaurant` executando `cargo new restaurant --lib`. Em seguida, insira o código da Listagem 7-1 em _src/lib.rs_ para definir alguns módulos e assinaturas de funções; este código é a seção de front of house.

**Arquivo: src/lib.rs**

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

[Listagem 7-1](#listagem-7-1): Um módulo `front_of_house` contendo outros módulos que então contêm funções

Definimos um módulo com a palavra-chave `mod` seguida do nome do módulo (neste caso, `front_of_house`). O corpo do módulo vai dentro de chaves. Dentro de módulos, podemos colocar outros módulos, como neste caso com os módulos `hosting` e `serving`. Módulos também podem conter definições de outros itens, como structs, enums, constantes, traits e, como na Listagem 7-1, funções.

Usando módulos, podemos agrupar definições relacionadas e nomear por que estão relacionadas. Programadores que usam este código podem navegar pelo código com base nos grupos em vez de precisar ler todas as definições, facilitando encontrar as definições relevantes para eles. Programadores que adicionam nova funcionalidade a este código saberão onde colocar o código para manter o programa organizado.

Mencionamos antes que _src/main.rs_ e _src/lib.rs_ são chamados de _raízes do crate_. O motivo do nome é que o conteúdo de qualquer um desses dois arquivos forma um módulo chamado `crate` na raiz da estrutura de módulos do crate, conhecida como _árvore de módulos_.

A Listagem 7-2 mostra a árvore de módulos para a estrutura da Listagem 7-1.

[Listagem 7-2](#listagem-7-2): A árvore de módulos para o código da Listagem 7-1

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

Esta árvore mostra como alguns módulos se aninham dentro de outros; por exemplo, `hosting` se aninha dentro de `front_of_house`. A árvore também mostra que alguns módulos são _irmãos_, o que significa que estão definidos no mesmo módulo; `hosting` e `serving` são irmãos definidos dentro de `front_of_house`. Se o módulo A está contido dentro do módulo B, dizemos que o módulo A é _filho_ do módulo B e que o módulo B é _pai_ do módulo A. Observe que toda a árvore de módulos está enraizada sob o módulo implícito chamado `crate`.

A árvore de módulos pode lembrá-lo da árvore de diretórios do sistema de arquivos no seu computador; essa é uma comparação muito adequada! Assim como diretórios em um sistema de arquivos, você usa módulos para organizar seu código. E assim como arquivos em um diretório, precisamos de uma forma de encontrar nossos módulos.
