---
title: "Controlar escopo e privacidade com módulos"
chapter_code: 07-02
slug: definindo-modulos-para-controlar-escopo-e-privacidade
challenge_day: 8
reading_minutes: 10
---

# Controlar Escopo e Privacidade com Módulos

Nesta seção, vamos falar sobre módulos e outras partes do sistema de módulos, como os _caminhos_, que permitem nomear itens; a palavra-chave `use`, que traz um caminho para o escopo; e a palavra-chave `pub`, usada para tornar itens públicos. Também vamos discutir a palavra-chave `as`, packages externos e o operador glob.

### Guia rápido de módulos

Antes de entrarmos nos detalhes de módulos e caminhos, aqui vai uma referência rápida de como módulos, caminhos, a palavra-chave `use` e a palavra-chave `pub` funcionam no compilador, e de como a maioria dos desenvolvedores organiza o código. Vamos passar por exemplos de cada uma dessas regras ao longo do capítulo, mas esta seção é um ótimo ponto de consulta para relembrar como módulos funcionam.

- **Comece pela raiz do crate**: ao compilar um crate, o compilador primeiro procura no arquivo de raiz do crate (geralmente _src/lib.rs_ para um library crate e _src/main.rs_ para um binary crate) o código a ser compilado.
- **Declarando módulos**: no arquivo de raiz do crate, você pode declarar novos módulos; por exemplo, um módulo "garden" com `mod garden;`. O compilador vai procurar o código do módulo nestes lugares:
  - Inline, dentro de chaves que substituem o ponto e vírgula após `mod garden`
  - No arquivo _src/garden.rs_
  - No arquivo _src/garden/mod.rs_
- **Declarando submódulos**: em qualquer arquivo que não seja a raiz do crate, você pode declarar submódulos. Por exemplo, pode declarar `mod vegetables;` em _src/garden.rs_. O compilador vai procurar o código do submódulo no diretório nomeado a partir do módulo pai nestes lugares:
  - Inline, diretamente após `mod vegetables`, dentro de chaves em vez do ponto e vírgula
  - No arquivo _src/garden/vegetables.rs_
  - No arquivo _src/garden/vegetables/mod.rs_
- **Caminhos para código em módulos**: depois que um módulo passa a fazer parte do seu crate, você pode referenciar código desse módulo de qualquer outro lugar no mesmo crate, desde que as regras de privacidade permitam, usando o caminho do código. Por exemplo, um tipo `Asparagus` no módulo `garden::vegetables` seria encontrado em `crate::garden::vegetables::Asparagus`.
- **Privado vs. público**: por padrão, o código dentro de um módulo é privado para seus módulos pais. Para tornar um módulo público, declare-o com `pub mod` em vez de `mod`. Para tornar públicos também os itens dentro de um módulo público, use `pub` antes de suas declarações.
- **A palavra-chave `use`**: dentro de um escopo, a palavra-chave `use` cria atalhos para itens, reduzindo a repetição de caminhos longos. Em qualquer escopo que possa referenciar `crate::garden::vegetables::Asparagus`, você pode criar um atalho com `use crate::garden::vegetables::Asparagus;` e, a partir daí, basta escrever `Asparagus` para usar esse tipo no escopo.

Aqui, criamos um binary crate chamado `backyard` para ilustrar essas regras. O diretório do crate, também chamado _backyard_, contém estes arquivos e diretórios:

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

Neste caso, o arquivo de raiz do crate é _src/main.rs_, e ele contém:

**Arquivo: src/main.rs**

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {plant:?}!");
}
```

A linha `pub mod garden;` diz ao compilador para incluir o código encontrado em _src/garden.rs_, que é:

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

Agora vamos entrar nos detalhes dessas regras e vê-las em ação!

### Agrupando código relacionado em módulos

_Módulos_ nos permitem organizar código dentro de um crate para melhorar a legibilidade e facilitar a reutilização. Módulos também nos permitem controlar a _privacidade_ dos itens, porque o código dentro de um módulo é privado por padrão. Itens privados são detalhes de implementação interna, não disponíveis para uso externo. Podemos escolher tornar públicos os módulos e os itens dentro deles, expondo-os para que código externo possa usá-los e depender deles.

Como exemplo, vamos escrever um library crate que fornece a funcionalidade de um restaurante. Vamos definir as assinaturas das funções, mas deixar seus corpos vazios para focar na organização do código, e não na implementação de um restaurante.

No setor de restaurantes, algumas áreas são chamadas de _front of house_ (salão) e outras de _back of house_ (cozinha). O _front of house_ é onde os clientes ficam: inclui a área em que os anfitriões os acomodam, os garçons anotam pedidos e recebem pagamentos, e os bartenders preparam bebidas. O _back of house_ é onde chefs e cozinheiros trabalham na cozinha, lava-louças fazem a limpeza e gerentes cuidam de tarefas administrativas.

Para estruturar nosso crate dessa forma, podemos organizar suas funções em módulos aninhados. Crie uma nova biblioteca chamada `restaurant` executando `cargo new restaurant --lib`. Em seguida, insira o código da Listagem 7-1 em _src/lib.rs_ para definir alguns módulos e assinaturas de função; esse código representa a seção de front of house.

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

<a id="listagem-7-1"></a>

[Listagem 7-1](#listagem-7-1): Um módulo `front_of_house` contendo outros módulos que então contêm funções

Definimos um módulo com a palavra-chave `mod`, seguida do nome do módulo (neste caso, `front_of_house`). O corpo do módulo fica dentro de chaves. Dentro de módulos, podemos colocar outros módulos, como neste caso com `hosting` e `serving`. Módulos também podem conter definições de outros itens, como structs, enums, constantes, traits e, como na Listagem 7-1, funções.

Usando módulos, podemos agrupar definições relacionadas e dar nomes que expressem essa relação. Programadores que usam esse código podem navegar por ele com base nesses grupos, em vez de precisar ler todas as definições, o que facilita encontrar o que é relevante para cada caso. Programadores que adicionarem novas funcionalidades também saberão onde colocar o código para manter o programa organizado.

Mencionamos antes que _src/main.rs_ e _src/lib.rs_ são chamados de _raízes do crate_. Esse nome existe porque o conteúdo de qualquer um desses arquivos forma um módulo chamado `crate`, na raiz da estrutura de módulos do crate, conhecida como _árvore de módulos_.

A Listagem 7-2 mostra a árvore de módulos da estrutura apresentada na Listagem 7-1.

<a id="listagem-7-2"></a>

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
[Listagem 7-2](#listagem-7-2): A árvore de módulos para o código da Listagem 7-1


Essa árvore mostra como alguns módulos se aninham dentro de outros; por exemplo, `hosting` está aninhado em `front_of_house`. A árvore também mostra que alguns módulos são _irmãos_, isto é, definidos dentro do mesmo módulo: `hosting` e `serving` são irmãos definidos dentro de `front_of_house`. Se o módulo A está contido no módulo B, dizemos que o módulo A é _filho_ do módulo B e que o módulo B é _pai_ do módulo A. Observe que toda a árvore de módulos está enraizada sob o módulo implícito chamado `crate`.

A árvore de módulos pode lembrar a árvore de diretórios do sistema de arquivos no seu computador, e essa comparação é muito adequada! Assim como diretórios em um sistema de arquivos, você usa módulos para organizar seu código. E, assim como usamos caminhos para encontrar arquivos em um diretório, também precisamos de uma forma de encontrar nossos módulos.
