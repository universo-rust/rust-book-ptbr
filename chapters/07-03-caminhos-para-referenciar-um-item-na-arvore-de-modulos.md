---
title: "Caminhos para referenciar um item na árvore de módulos"
chapter_code: 07-03
slug: caminhos-para-referenciar-um-item-na-arvore-de-modulos
---

# Caminhos para Referenciar um Item na Árvore de Módulos

Para mostrar ao Rust onde encontrar um item na árvore de módulos, usamos um caminho da mesma forma que usamos caminhos ao navegar em um sistema de arquivos. Para chamar uma função, precisamos conhecer seu caminho.

Um caminho pode ter duas formas:

- Um _caminho absoluto_ é o caminho completo que começa na raiz de um crate; para código de um crate externo, o caminho absoluto começa com o nome do crate e, para código do crate atual, começa com o literal `crate`.
- Um _caminho relativo_ começa no módulo atual e usa `self`, `super` ou um identificador do módulo atual.

Tanto caminhos absolutos quanto relativos são formados por um ou mais identificadores separados por dois-pontos duplos (`::`).

Voltando à Listagem 7-1, digamos que queremos chamar a função `add_to_waitlist`. Isso equivale a perguntar: qual é o caminho da função `add_to_waitlist`? A Listagem 7-3 mostra a Listagem 7-1 com alguns módulos e funções removidos.

Vamos mostrar duas formas de chamar a função `add_to_waitlist` a partir de uma nova função, `eat_at_restaurant`, definida na raiz do crate. Esses caminhos estão corretos, mas ainda existe outro problema que impede esse exemplo de compilar como está. Vamos explicar o motivo em seguida.

A função `eat_at_restaurant` faz parte da API pública do nosso library crate, então a marcamos com a palavra-chave `pub`. Na seção [Expondo caminhos com a palavra-chave `pub`](/livro/cap07-03-caminhos-para-referenciar-um-item-na-arvore-de-modulos#expondo-caminhos-com-a-palavra-chave-pub), vamos entrar em mais detalhes sobre `pub`.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Caminho absoluto
    crate::front_of_house::hosting::add_to_waitlist();

    // Caminho relativo
    front_of_house::hosting::add_to_waitlist();
}
```

<a id="listagem-7-3"></a>

[Listagem 7-3](#listagem-7-3): Chamando a função `add_to_waitlist` usando caminhos absolutos e relativos

Na primeira vez em que chamamos a função `add_to_waitlist` em `eat_at_restaurant`, usamos um caminho absoluto. A função `add_to_waitlist` é definida no mesmo crate que `eat_at_restaurant`, então podemos usar a palavra-chave `crate` para iniciar um caminho absoluto. Em seguida, incluímos cada módulo sucessivo até chegar a `add_to_waitlist`. Você pode imaginar um sistema de arquivos com a mesma estrutura: especificaríamos o caminho `/front_of_house/hosting/add_to_waitlist` para executar o programa `add_to_waitlist`; usar `crate` para começar da raiz do crate é como usar `/` para começar da raiz do sistema de arquivos no shell.

Na segunda vez em que chamamos `add_to_waitlist` em `eat_at_restaurant`, usamos um caminho relativo. O caminho começa com `front_of_house`, o nome do módulo definido no mesmo nível da árvore de módulos que `eat_at_restaurant`. Aqui, o equivalente no sistema de arquivos seria usar o caminho `front_of_house/hosting/add_to_waitlist`. Começar com o nome de um módulo significa que o caminho é relativo.

Escolher entre caminho relativo e caminho absoluto é uma decisão que você tomará com base no seu projeto, e isso depende de ser mais provável mover o código que define um item separadamente do código que o usa, ou mover ambos juntos. Por exemplo, se movêssemos o módulo `front_of_house` e a função `eat_at_restaurant` para um módulo chamado `customer_experience`, precisaríamos atualizar o caminho absoluto para `add_to_waitlist`, mas o caminho relativo continuaria válido. Já se movêssemos apenas a função `eat_at_restaurant` para um módulo chamado `dining`, o caminho absoluto para `add_to_waitlist` permaneceria o mesmo, mas o caminho relativo precisaria ser atualizado. Em geral, preferimos caminhos absolutos porque é mais provável que queiramos mover definições e chamadas de itens independentemente umas das outras.

Vamos tentar compilar a Listagem 7-3 e descobrir por que ela ainda não compila! Os erros que obtemos são mostrados na Listagem 7-4.

<a id="listagem-7-4"></a>
```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^  --------------- function `add_to_waitlist` is not publicly re-exported
  |                            |
  |                            private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^  --------------- function `add_to_waitlist` is not publicly re-exported
   |                     |
   |                     private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
 2 |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` (lib) due to 2 previous errors
```
[Listagem 7-4](#listagem-7-4): Erros do compilador ao compilar o código da Listagem 7-3

As mensagens de erro dizem que o módulo `hosting` é privado. Em outras palavras, temos os caminhos corretos para o módulo `hosting` e para a função `add_to_waitlist`, mas o Rust não nos permite usá-los por causa das regras de privacidade. Em Rust, todos os itens (funções, métodos, structs, enums, módulos e constantes) são privados para os módulos pais por padrão. Se quiser tornar um item como função ou struct público, use a palavra-chave `pub`.

Itens em um módulo pai não podem usar os itens privados dentro de módulos filhos, mas itens em módulos filhos podem usar os itens de seus módulos ancestrais. Isso acontece porque módulos filhos encapsulam e escondem seus detalhes de implementação, mas ainda conseguem ver o contexto em que foram definidos. Para continuar a metáfora, pense nas regras de privacidade como os bastidores de um restaurante: o que acontece ali é privado para os clientes, mas os gerentes conseguem ver e fazer tudo no restaurante que administram.

Rust foi projetado para que esconder detalhes internos de implementação seja o comportamento padrão. Assim, você sabe quais partes do código interno pode alterar sem quebrar código externo. Ainda assim, Rust permite expor partes internas do código de módulos filhos para módulos ancestrais usando a palavra-chave `pub`, que torna um item público.

### Expondo caminhos com a palavra-chave `pub`

Vamos voltar ao erro da Listagem 7-4, que nos disse que o módulo `hosting` é privado. Queremos que a função `eat_at_restaurant`, no módulo pai, tenha acesso à função `add_to_waitlist`, no módulo filho; então marcamos o módulo `hosting` com a palavra-chave `pub`, como mostrado na Listagem 7-5.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Caminho absoluto
    crate::front_of_house::hosting::add_to_waitlist();

    // Caminho relativo
    front_of_house::hosting::add_to_waitlist();
}
```

<a id="listagem-7-5"></a>

[Listagem 7-5](#listagem-7-5): Declarando o módulo `hosting` como `pub` para usá-lo a partir de `eat_at_restaurant`

Infelizmente, o código da Listagem 7-5 ainda resulta em erros do compilador, como mostrado na Listagem 7-6.

<a id="listagem-7-6"></a>
```console
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:10:37
   |
10 |     crate::front_of_house::hosting::add_to_waitlist();
   |                                     ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
 3 |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:13:30
   |
13 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
 3 |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` (lib) due to 2 previous errors
```
[Listagem 7-6](#listagem-7-6): Erros do compilador ao compilar o código da Listagem 7-5

O que aconteceu? Adicionar `pub` antes de `mod hosting` torna o módulo público. Com essa mudança, se conseguimos acessar `front_of_house`, conseguimos acessar `hosting`. Mas o _conteúdo_ de `hosting` ainda continua privado; tornar o módulo público não torna seu conteúdo público automaticamente. O `pub` em um módulo apenas permite que código de seus módulos ancestrais o referencie, não que acesse seu código interno. Como módulos são contêineres, não há muito o que fazer apenas tornando o módulo público; precisamos ir além e tornar públicos também um ou mais itens dentro dele.

Os erros da Listagem 7-6 dizem que a função `add_to_waitlist` é privada. As regras de privacidade se aplicam a structs, enums, funções e métodos, bem como a módulos.

Vamos também tornar a função `add_to_waitlist` pública, adicionando a palavra-chave `pub` antes da sua definição, como na Listagem 7-7.

**Arquivo: src/lib.rs**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Caminho absoluto
    crate::front_of_house::hosting::add_to_waitlist();

    // Caminho relativo
    front_of_house::hosting::add_to_waitlist();
}
```

<a id="listagem-7-7"></a>

[Listagem 7-7](#listagem-7-7): Adicionar a palavra-chave `pub` em `mod hosting` e `fn add_to_waitlist` permite chamar a função a partir de `eat_at_restaurant`

Agora o código compila! Para entender por que adicionar `pub` nos permite usar esses caminhos em `eat_at_restaurant` dentro das regras de privacidade, vamos olhar o caminho absoluto e o relativo.

No caminho absoluto, começamos com `crate`, a raiz da árvore de módulos do nosso crate. O módulo `front_of_house` é definido na raiz do crate. Embora `front_of_house` não seja público, como a função `eat_at_restaurant` está definida no mesmo módulo que `front_of_house` (isto é, `eat_at_restaurant` e `front_of_house` são irmãos), podemos referenciar `front_of_house` a partir de `eat_at_restaurant`. Em seguida vem o módulo `hosting`, marcado com `pub`. Como conseguimos acessar o módulo pai de `hosting`, também conseguimos acessar `hosting`. Por fim, a função `add_to_waitlist` está marcada com `pub`, e como conseguimos acessar seu módulo pai, essa chamada de função funciona.

No caminho relativo, a lógica é a mesma do caminho absoluto, exceto no primeiro passo: em vez de começar na raiz do crate, o caminho começa em `front_of_house`. O módulo `front_of_house` é definido dentro do mesmo módulo que `eat_at_restaurant`, então o caminho relativo iniciando no módulo em que `eat_at_restaurant` está definido funciona. E, como `hosting` e `add_to_waitlist` estão marcados com `pub`, o restante do caminho também funciona, tornando essa chamada de função válida.

Se você planeja compartilhar seu library crate para que outros projetos usem seu código, sua API pública é o contrato com os usuários do seu crate e determina como eles podem interagir com ele. Existem várias considerações sobre como gerenciar mudanças nessa API pública para facilitar a vida de quem depende do seu crate. Essas considerações estão além do escopo deste livro; se você tiver interesse no tema, consulte as Rust API Guidelines.

> #### Boas práticas para packages com binário e biblioteca
>
> Mencionamos que um package pode conter tanto uma raiz de binary crate em _src/main.rs_ quanto uma raiz de library crate em _src/lib.rs_, e que ambos os crates terão o nome do package por padrão. Em geral, packages que seguem esse padrão (com library crate e binary crate) deixam no binary crate apenas o código necessário para iniciar um executável que chama o código definido no library crate. Isso permite que outros projetos aproveitem a maior parte da funcionalidade que o package oferece, porque o código do library crate pode ser compartilhado.
>
> A árvore de módulos deve ser definida em _src/lib.rs_. Assim, quaisquer itens públicos podem ser usados no binary crate iniciando caminhos com o nome do package. O binary crate se torna um usuário do library crate da mesma forma que um crate totalmente externo usaria esse library crate: ele só pode usar a API pública. Isso ajuda você a projetar uma boa API; você não é apenas o autor, mas também um cliente.
>
> No Capítulo 12, demonstraremos essa prática organizacional com um programa de linha de comando que conterá tanto um binary crate quanto um library crate.

### Iniciando caminhos relativos com `super`

Podemos construir caminhos relativos que começam no módulo pai, em vez do módulo atual ou da raiz do crate, usando `super` no início do caminho. Isso é como iniciar um caminho no sistema de arquivos com a sintaxe `..`, que significa ir para o diretório pai. Usar `super` nos permite referenciar um item que sabemos estar no módulo pai, o que pode facilitar a reorganização da árvore de módulos quando o módulo está intimamente relacionado ao pai, mas o pai pode ser movido para outro lugar da árvore algum dia.

Considere o código da Listagem 7-8, que modela a situação em que um chef corrige um pedido incorreto e o leva pessoalmente ao cliente. A função `fix_incorrect_order`, definida no módulo `back_of_house`, chama a função `deliver_order`, definida no módulo pai, especificando o caminho para `deliver_order` iniciado com `super`.

**Arquivo: src/lib.rs**

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

<a id="listagem-7-8"></a>

[Listagem 7-8](#listagem-7-8): Chamando uma função usando um caminho relativo começando com `super`

A função `fix_incorrect_order` está no módulo `back_of_house`, então podemos usar `super` para subir ao módulo pai de `back_of_house`, que neste caso é `crate`, a raiz. A partir daí, procuramos `deliver_order` e a encontramos. Sucesso! Entendemos que o módulo `back_of_house` e a função `deliver_order` provavelmente permanecerão na mesma relação e serão movidos juntos caso decidamos reorganizar a árvore de módulos do crate. Por isso, usamos `super`, para ter menos lugares para atualizar no futuro se esse código for movido para outro módulo.

### Tornando structs e enums públicos

Também podemos usar `pub` para tornar structs e enums públicos, mas existem alguns detalhes extras no uso de `pub` com esses tipos. Se usarmos `pub` antes da definição de uma struct, tornamos a struct pública, mas seus campos continuam privados. Podemos decidir caso a caso se cada campo será público ou não. Na Listagem 7-9, definimos uma struct pública `back_of_house::Breakfast` com um campo público, `toast`, e um campo privado, `seasonal_fruit`. Isso modela o caso de um restaurante em que o cliente pode escolher o tipo de pão da refeição, mas o chef decide qual fruta a acompanha com base no que está na estação e em estoque. Como a fruta disponível muda rápido, os clientes não podem escolher a fruta, nem mesmo ver qual receberão.

**Arquivo: src/lib.rs**

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("pêssegos"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Pedir um café da manhã no verão com torrada de centeio.
    let mut meal = back_of_house::Breakfast::summer("Centeio");
    // Mudar de ideia sobre qual pão queremos.
    meal.toast = String::from("Trigo");
    println!("Quero uma torrada de {}, por favor", meal.toast);

    // A próxima linha não compilará se descomentarmos; não temos permissão
    // para ver ou modificar a fruta da estação que vem com a refeição.
    // meal.seasonal_fruit = String::from("mirtilos");
}
```

<a id="listagem-7-9"></a>

[Listagem 7-9](#listagem-7-9): Uma struct com alguns campos públicos e alguns privados

Como o campo `toast` na struct `back_of_house::Breakfast` é público, em `eat_at_restaurant` podemos escrever e ler esse campo usando notação de ponto. Observe que não podemos usar o campo `seasonal_fruit` em `eat_at_restaurant`, porque `seasonal_fruit` é privado. Tente descomentar a linha que modifica o valor de `seasonal_fruit` para ver qual erro aparece.

Além disso, note que, como `back_of_house::Breakfast` tem um campo privado, a struct precisa fornecer uma função associada pública que construa uma instância de `Breakfast` (aqui, ela se chama `summer`). Se `Breakfast` não tivesse essa função, não conseguiríamos criar uma instância de `Breakfast` em `eat_at_restaurant`, porque não poderíamos definir o valor do campo privado `seasonal_fruit` nesse contexto.

Em contraste, se tornarmos um enum público, todas as suas variantes também se tornam públicas. Só precisamos de `pub` antes da palavra-chave `enum`, como mostrado na Listagem 7-10.

**Arquivo: src/lib.rs**

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

<a id="listagem-7-10"></a>

[Listagem 7-10](#listagem-7-10): Designar um enum como público torna todas as suas variantes públicas

Como tornamos o enum `Appetizer` público, podemos usar as variantes `Soup` e `Salad` em `eat_at_restaurant`.

Enums não são muito úteis a menos que suas variantes sejam públicas; seria incômodo ter que anotar todas as variantes com `pub` em todos os casos, então o padrão para variantes de enum é serem públicas. Structs, por outro lado, costumam ser úteis mesmo com campos privados; por isso, campos de struct seguem a regra geral de tudo ser privado por padrão, a menos que seja anotado com `pub`.

Há mais uma situação envolvendo `pub` que ainda não cobrimos, e ela envolve o último recurso do sistema de módulos: a palavra-chave `use`. Primeiro vamos cobrir `use` isoladamente e, depois, mostrar como combinar `pub` e `use`.
