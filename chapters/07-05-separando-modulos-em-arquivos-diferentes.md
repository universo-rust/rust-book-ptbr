---
title: "Separando módulos em arquivos diferentes"
chapter_code: 07-05
slug: separando-modulos-em-arquivos-diferentes
challenge_day: 9
reading_minutes: 5
---

# Separando módulos em arquivos diferentes

Até agora, todos os exemplos deste capítulo definiam vários módulos em um único arquivo. Quando os módulos crescem, você pode querer mover suas definições para arquivos separados, facilitando a navegação no código.

Por exemplo, vamos começar com o código da Listagem 7-17, que tinha vários módulos do restaurante. Vamos extrair esses módulos para arquivos, em vez de deixar tudo definido no arquivo de raiz do crate. Neste caso, o arquivo de raiz do crate é _src/lib.rs_, mas esse procedimento também funciona com binary crates cuja raiz é _src/main.rs_.

Primeiro, vamos extrair o módulo `front_of_house` para um arquivo próprio. Remova o código dentro das chaves do módulo `front_of_house`, deixando apenas a declaração `mod front_of_house;`, para que _src/lib.rs_ fique com o código mostrado na Listagem 7-21. Observe que isso não vai compilar até criarmos o arquivo _src/front_of_house.rs_, mostrado na Listagem 7-22.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

<a id="listagem-7-21"></a>

[Listagem 7-21](#listagem-7-21): Declarando o módulo `front_of_house` cujo corpo estará em _src/front_of_house.rs_

Em seguida, coloque em um novo arquivo chamado _src/front_of_house.rs_ o código que estava dentro das chaves, como mostrado na Listagem 7-22. O compilador sabe que deve procurar nesse arquivo porque encontrou, na raiz do crate, a declaração de módulo com o nome `front_of_house`.

**Arquivo: src/front_of_house.rs**

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

<a id="listagem-7-22"></a>

[Listagem 7-22](#listagem-7-22): Definições dentro do módulo `front_of_house` em _src/front_of_house.rs_

Observe que você só precisa carregar um arquivo com uma declaração `mod` _uma vez_ na árvore de módulos. Depois que o compilador sabe que o arquivo faz parte do projeto (e sabe onde ele está na árvore de módulos por causa de onde você colocou a declaração `mod`), os outros arquivos do projeto devem referenciar esse código por meio do caminho onde ele foi declarado, como vimos na seção [Caminhos para referenciar um item na árvore de módulos](/livro/cap07-03-caminhos-para-referenciar-um-item-na-arvore-de-modulos). Em outras palavras, `mod` _não_ é uma operação de "include", como em outras linguagens de programação.

Em seguida, vamos extrair o módulo `hosting` para um arquivo próprio. O processo é um pouco diferente porque `hosting` é um módulo filho de `front_of_house`, e não do módulo raiz. Vamos colocar o arquivo de `hosting` em um novo diretório nomeado de acordo com seus ancestrais na árvore de módulos; neste caso, _src/front_of_house_.

Para começar a mover `hosting`, alteramos _src/front_of_house.rs_ para conter apenas a declaração do módulo `hosting`:

**Arquivo: src/front_of_house.rs**

```rust
pub mod hosting;
```

Depois, criamos o diretório _src/front_of_house_ e o arquivo _hosting.rs_ para conter as definições do módulo `hosting`:

**Arquivo: src/front_of_house/hosting.rs**

```rust
pub fn add_to_waitlist() {}
```

Se colocássemos _hosting.rs_ no diretório _src_, o compilador esperaria que o código de _hosting.rs_ estivesse em um módulo `hosting` declarado na raiz do crate, e não como filho do módulo `front_of_house`. As regras do compilador sobre quais arquivos verificar para o código de cada módulo fazem com que diretórios e arquivos reflitam de perto a árvore de módulos.

> ### Caminhos de arquivo alternativos
>
> Até agora, cobrimos os caminhos de arquivo mais idiomáticos usados pelo compilador Rust, mas o Rust também oferece suporte a um estilo mais antigo de caminho de arquivo. Para um módulo chamado `front_of_house` declarado na raiz do crate, o compilador vai procurar o código do módulo em:
>
> - _src/front_of_house.rs_ (o que cobrimos)
> - _src/front_of_house/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Para um módulo chamado `hosting`, que é submódulo de `front_of_house`, o compilador vai procurar o código do módulo em:
>
> - _src/front_of_house/hosting.rs_ (o que cobrimos)
> - _src/front_of_house/hosting/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Se você usar os dois estilos para o mesmo módulo, receberá um erro de compilação. Usar uma mistura dos dois estilos para módulos diferentes no mesmo projeto é permitido, mas pode confundir quem estiver navegando pelo projeto.
>
> A principal desvantagem do estilo que usa arquivos chamados _mod.rs_ é que o projeto pode acabar com muitos arquivos com esse mesmo nome, o que pode ficar confuso quando vários deles estão abertos no editor ao mesmo tempo.

Movemos o código de cada módulo para um arquivo separado, e a árvore de módulos continua a mesma. As chamadas de função em `eat_at_restaurant` funcionam sem nenhuma modificação, mesmo com as definições em arquivos diferentes. Essa técnica permite mover módulos para novos arquivos conforme eles crescem.

Observe que a declaração `pub use crate::front_of_house::hosting` em _src/lib.rs_ também não mudou, e `use` não tem qualquer impacto sobre quais arquivos são compilados como parte do crate. A palavra-chave `mod` declara módulos, e o Rust procura, em um arquivo com o mesmo nome do módulo, o código que pertence àquele módulo.

## Resumo

Rust permite dividir um package em vários crates e um crate em módulos, para que você possa referenciar, em um módulo, itens definidos em outro. Você pode fazer isso especificando caminhos absolutos ou relativos. Esses caminhos podem ser trazidos para o escopo com uma declaração `use`, permitindo usar um caminho mais curto quando o item for usado várias vezes no mesmo escopo. O código de módulo é privado por padrão, mas você pode tornar definições públicas adicionando a palavra-chave `pub`.

No próximo capítulo, veremos algumas estruturas de dados de coleção da biblioteca padrão que você pode usar no seu código bem organizado.
