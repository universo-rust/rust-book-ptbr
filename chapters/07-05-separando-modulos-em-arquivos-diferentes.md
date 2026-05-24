---
title: "Separando módulos em arquivos diferentes"
chapter_code: 07-05
slug: separando-modulos-em-arquivos-diferentes
---

# Separando Módulos em Arquivos Diferentes

Até agora, todos os exemplos deste capítulo definiram vários módulos em um arquivo. Quando os módulos ficam grandes, você pode querer mover suas definições para um arquivo separado para facilitar a navegação no código.

Por exemplo, vamos começar do código da Listagem 7-17 que tinha vários módulos de restaurante. Extrairemos módulos para arquivos em vez de ter todos os módulos definidos no arquivo de raiz do crate. Neste caso, o arquivo de raiz do crate é _src/lib.rs_, mas este procedimento também funciona com binary crates cuja raiz do crate é _src/main.rs_.

Primeiro, extrairemos o módulo `front_of_house` para seu próprio arquivo. Remova o código dentro das chaves do módulo `front_of_house`, deixando apenas a declaração `mod front_of_house;`, para que _src/lib.rs_ contenha o código mostrado na Listagem 7-21. Observe que isso não compilará até criarmos o arquivo _src/front_of_house.rs_ na Listagem 7-22.

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

Em seguida, coloque o código que estava nas chaves em um novo arquivo chamado _src/front_of_house.rs_, como mostrado na Listagem 7-22. O compilador sabe procurar neste arquivo porque encontrou a declaração do módulo na raiz do crate com o nome `front_of_house`.

**Arquivo: src/front_of_house.rs**

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

<a id="listagem-7-22"></a>

[Listagem 7-22](#listagem-7-22): Definições dentro do módulo `front_of_house` em _src/front_of_house.rs_

Observe que você só precisa carregar um arquivo usando uma declaração `mod` _uma vez_ na sua árvore de módulos. Uma vez que o compilador sabe que o arquivo faz parte do projeto (e sabe onde na árvore de módulos o código reside por causa de onde você colocou a declaração `mod`), outros arquivos no seu projeto devem referenciar o código do arquivo carregado usando um caminho para onde foi declarado, como coberto na seção Caminhos para referenciar um item na árvore de módulos. Em outras palavras, `mod` _não_ é uma operação de “include” que você pode ter visto em outras linguagens de programação.

Em seguida, extrairemos o módulo `hosting` para seu próprio arquivo. O processo é um pouco diferente porque `hosting` é um módulo filho de `front_of_house`, não do módulo raiz. Colocaremos o arquivo para `hosting` em um novo diretório que será nomeado para seus ancestrais na árvore de módulos, neste caso _src/front_of_house_.

Para começar a mover `hosting`, alteramos _src/front_of_house.rs_ para conter apenas a declaração do módulo `hosting`:

**Arquivo: src/front_of_house.rs**

```rust
pub mod hosting;
```

Em seguida, criamos um diretório _src/front_of_house_ e um arquivo _hosting.rs_ para conter as definições feitas no módulo `hosting`:

**Arquivo: src/front_of_house/hosting.rs**

```rust
pub fn add_to_waitlist() {}
```

Se colocássemos _hosting.rs_ no diretório _src_, o compilador esperaria que o código de _hosting.rs_ estivesse em um módulo `hosting` declarado na raiz do crate e não declarado como filho do módulo `front_of_house`. As regras do compilador sobre quais arquivos verificar para o código de quais módulos significam que diretórios e arquivos correspondem mais de perto à árvore de módulos.

> ### Caminhos de arquivo alternativos
>
> Até agora cobrimos os caminhos de arquivo mais idiomáticos que o compilador Rust usa, mas o Rust também suporta um estilo mais antigo de caminho de arquivo. Para um módulo chamado `front_of_house` declarado na raiz do crate, o compilador procurará o código do módulo em:
>
> - _src/front_of_house.rs_ (o que cobrimos)
> - _src/front_of_house/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Para um módulo chamado `hosting` que é submódulo de `front_of_house`, o compilador procurará o código do módulo em:
>
> - _src/front_of_house/hosting.rs_ (o que cobrimos)
> - _src/front_of_house/hosting/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Se você usar ambos os estilos para o mesmo módulo, obterá um erro do compilador. Usar uma mistura de ambos os estilos para módulos diferentes no mesmo projeto é permitido, mas pode ser confuso para pessoas navegando no seu projeto.
>
> A principal desvantagem do estilo que usa arquivos chamados _mod.rs_ é que seu projeto pode acabar com muitos arquivos chamados _mod.rs_, o que pode ficar confuso quando você os tem abertos no editor ao mesmo tempo.

Movemos o código de cada módulo para um arquivo separado, e a árvore de módulos permanece a mesma. As chamadas de função em `eat_at_restaurant` funcionarão sem nenhuma modificação, mesmo que as definições vivam em arquivos diferentes. Essa técnica permite mover módulos para novos arquivos conforme crescem em tamanho.

Observe que a declaração `pub use crate::front_of_house::hosting` em _src/lib.rs_ também não mudou, nem `use` tem qualquer impacto sobre quais arquivos são compilados como parte do crate. A palavra-chave `mod` declara módulos, e o Rust procura em um arquivo com o mesmo nome do módulo o código que vai naquele módulo.

## Resumo

O Rust permite dividir um package em vários crates e um crate em módulos, para que você possa referenciar itens definidos em um módulo a partir de outro módulo. Você pode fazer isso especificando caminhos absolutos ou relativos. Esses caminhos podem ser trazidos para o escopo com uma declaração `use` para que você possa usar um caminho mais curto para vários usos do item nesse escopo. O código de módulo é privado por padrão, mas você pode tornar definições públicas adicionando a palavra-chave `pub`.

No próximo capítulo, veremos algumas estruturas de dados de coleção na biblioteca padrão que você pode usar no seu código bem organizado.
