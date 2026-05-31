---
title: "Definindo e instanciando structs"
chapter_code: 05-01
slug: definindo-structs
---

# Definindo e Instanciando Structs

Structs são semelhantes às tuplas, discutidas na seção [O tipo tupla (tuple)](/livro/cap03-02-tipos-de-dados#o-tipo-tupla-tuple), pois ambas armazenam múltiplos valores relacionados. Assim como nas tuplas, os elementos de uma struct podem ter tipos diferentes. No entanto, diferentemente das tuplas, em uma struct você dá um nome para cada parte dos dados, tornando claro o significado de cada valor. Esses nomes tornam as structs mais flexíveis do que as tuplas: você não precisa depender da ordem dos dados para especificar ou acessar os valores de uma instância.

Para definir uma struct, utilizamos a palavra-chave `struct` seguida do nome da struct. O nome da struct deve descrever o significado do conjunto de dados que está sendo agrupado. Em seguida, dentro de chaves (`{}`), definimos os nomes e os tipos dos dados que ela contém, chamados de _campos_ (_fields_). Por exemplo, a Listagem 5-1 mostra uma struct que armazena informações sobre uma conta de usuário.

**Arquivo: src/main.rs**

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

<a id="listagem-5-1"></a>

[Listagem 5-1](#listagem-5-1): Uma definição de struct `User`

Para utilizar uma struct após defini-la, criamos uma _instância_ dessa struct fornecendo valores concretos para cada um de seus campos. Criamos uma instância informando o nome da struct seguido de chaves contendo pares _`campo: valor`_, onde os campos são os nomes definidos na struct e os valores são os dados que desejamos armazenar neles.

Não é necessário especificar os campos na mesma ordem em que foram declarados na struct. Em outras palavras, a definição da struct funciona como um modelo geral para o tipo, e as instâncias preenchem esse modelo com dados específicos para criar valores desse tipo.

Por exemplo, podemos declarar um usuário específico conforme mostrado na Listagem 5-2.

**Arquivo: src/main.rs**

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

<a id="listagem-5-2"></a>

[Listagem 5-2](#listagem-5-2): Criando uma instância da struct `User`

Para obter um valor específico de uma struct, usamos a notação de ponto (dot notation). Por exemplo, para acessar o endereço de e-mail desse usuário, usamos `user1.email`. Se a instância for mutável, podemos alterar um valor utilizando a notação de ponto e atribuindo um novo valor a um campo específico. A Listagem 5-3 mostra como modificar o valor armazenado no campo `email` de uma instância mutável de `User`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

<a id="listagem-5-3"></a>

[Listagem 5-3](#listagem-5-3): Alterando o valor do campo `email` de uma instância de `User`

Note que toda a instância precisa ser mutável; o Rust não permite que marquemos apenas alguns campos como mutáveis. Assim como em qualquer expressão, podemos construir uma nova instância da struct como a última expressão no corpo da função para retorná-la implicitamente.

A Listagem 5-4 mostra uma função `build_user` que retorna uma instância de `User` com o email e o nome de usuário fornecidos. O campo `active` recebe o valor `true`, e `sign_in_count` recebe o valor `1`.

**Arquivo: src/main.rs**

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

<a id="listagem-5-4"></a>

[Listagem 5-4](#listagem-5-4): Uma função `build_user` que recebe e-mail e nome de usuário e retorna uma instância de `User`

Faz sentido nomear os parâmetros da função com os mesmos nomes dos campos da struct, mas ter que repetir os nomes dos campos `email` e `username` junto com as variáveis é um pouco trabalhoso. Se a struct tivesse mais campos, repetir cada nome ficaria ainda mais irritante. Felizmente, existe uma forma abreviada conveniente!

### Usando a forma abreviada de inicialização de campos

Como os nomes dos parâmetros e os nomes dos campos da struct são exatamente os mesmos na Listagem 5-4, podemos usar a sintaxe abreviada de inicialização de campos (field init shorthand) para reescrever `build_user` de forma que ela tenha exatamente o mesmo comportamento, mas sem a repetição de `username` e `email`, como mostrado na Listagem 5-5.

**Arquivo: src/main.rs**

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

<a id="listagem-5-5"></a>

[Listagem 5-5](#listagem-5-5): Uma função `build_user` que usa a forma abreviada de inicialização de campo porque os parâmetros `username` e `email` têm o mesmo nome que os campos da estrutura.

Aqui, estamos criando uma nova instância da struct `User`, que possui um campo chamado `email`. Queremos definir o valor do campo `email` como o valor do parâmetro `email` da função `build_user`. Como o campo `email` e o parâmetro `email` têm o mesmo nome, precisamos apenas escrever `email`, em vez de `email: email`.

### Criando instâncias com sintaxe de atualização de struct

Muitas vezes é útil criar uma nova instância de uma struct que aproveite a maioria dos valores de outra instância do mesmo tipo, mas altere alguns deles. Você pode fazer isso usando a sintaxe de atualização de struct.

Primeiro, na Listagem 5-6, mostramos como criar uma nova instância de `User` em `user2` da forma tradicional, sem usar a sintaxe de atualização. Definimos um novo valor para `email`, mas, caso contrário, usamos os mesmos valores de `user1` que criamos na Listagem 5-2.

**Arquivo: src/main.rs**

```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```

<a id="listagem-5-6"></a>

[Listagem 5-6](#listagem-5-6): Criando uma nova instância de `User` usando todos os valores de `user1`, exceto um

Usando a sintaxe de atualização de struct, podemos obter o mesmo efeito com menos código, como mostrado na Listagem 5-7. A sintaxe `..` indica que os campos restantes, não definidos explicitamente, devem receber os mesmos valores dos campos correspondentes na instância fornecida.

**Arquivo: src/main.rs**

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

<a id="listagem-5-7"></a>

[Listagem 5-7](#listagem-5-7): Usando a sintaxe de atualização de struct para definir um novo valor de `email` em uma instância de `User`, reutilizando o restante dos valores de `user1`

O código da Listagem 5-7 também cria uma instância em `user2` com um valor diferente para `email`, mas com os mesmos valores dos campos `username`, `active` e `sign_in_count` de `user1`. O `..user1` deve vir por último para indicar que os campos restantes devem obter seus valores dos campos correspondentes em `user1`; no entanto, podemos especificar valores para quantos campos quisermos, em qualquer ordem, independentemente da ordem dos campos na definição da struct.

Observe que a sintaxe de atualização de struct usa `=` como em uma atribuição; isso ocorre porque ela move os dados, como vimos na seção [Variáveis e dados interagindo com move](/livro/cap04-01-o-que-e-ownership#variáveis-e-dados-interagindo-com-move) do Capítulo 4. Neste exemplo, não podemos mais usar `user1` depois de criar `user2`, porque a `String` no campo `username` de `user1` foi movida para `user2`. Se tivéssemos atribuído a `user2` novos valores `String` tanto para `email` quanto para `username` — e, assim, usado apenas os valores de `active` e `sign_in_count` de `user1` —, então `user1` ainda seria válida depois de criar `user2`. Tanto `active` quanto `sign_in_count` são tipos que implementam a trait `Copy`, de modo que se aplicaria o comportamento discutido na seção [Dados somente na pilha: Copy](/livro/cap04-01-o-que-e-ownership#dados-somente-na-pilha-copy) do Capítulo 4. Também poderíamos usar `user1.email` neste exemplo, porque esse valor não foi movido para fora de `user1`.

### Criando tipos diferentes com tuple structs

O Rust também suporta structs semelhantes a tuplas, chamadas de _tuple structs_. As tuple structs trazem o significado adicional que o nome da struct oferece, mas não têm nomes associados aos campos; em vez disso, possuem apenas os tipos dos campos. Tuple structs são úteis quando você quer dar um nome à tupla inteira e torná-la um tipo diferente das demais tuplas, e quando nomear cada campo como em uma struct comum seria verboso ou redundante.

Para definir uma tuple struct, comece com a palavra-chave `struct` e o nome da struct, seguidos pelos tipos na tupla. Por exemplo, aqui definimos e usamos duas tuple structs chamadas `Color` e `Point`:

**Arquivo: src/main.rs**

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

Observe que os valores `black` e `origin` são de tipos diferentes porque são instâncias de tuple structs distintas. Cada struct que você define é seu próprio tipo, mesmo que os campos dentro da struct possam ter os mesmos tipos. Por exemplo, uma função que recebe um parâmetro do tipo `Color` não pode receber um `Point` como argumento, mesmo que ambos os tipos sejam compostos por três valores `i32`. Fora isso, instâncias de tuple struct são semelhantes a tuplas no sentido de que você pode desestruturá-las em partes individuais e pode usar um `.` seguido do índice para acessar um valor individual. Diferentemente das tuplas, tuple structs exigem que você informe o tipo da struct ao desestruturá-las. Por exemplo, escreveríamos `let Point(x, y, z) = origin;` para desestruturar os valores do ponto `origin` em variáveis chamadas `x`, `y` e `z`.

### Definindo unit-like structs

Você também pode definir structs que não têm nenhum campo! Essas são chamadas de _unit-like structs_, porque se comportam de forma semelhante a `()`, o tipo unitário que mencionamos na seção [O Tipo Tupla](/livro/cap03-02-tipos-de-dados#o-tipo-tupla-tuple). Unit-like structs podem ser úteis quando você precisa implementar uma trait em algum tipo, mas não tem dados que queira armazenar no próprio tipo. Falaremos sobre traits no Capítulo 10. Aqui está um exemplo de declaração e instanciação de uma unit struct chamada `AlwaysEqual`:

**Arquivo: src/main.rs**

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

Para definir `AlwaysEqual`, usamos a palavra-chave `struct`, o nome desejado e, em seguida, um ponto e vírgula. Não é necessário usar chaves ou parênteses! Depois, podemos obter uma instância de `AlwaysEqual` na variável `subject` de forma semelhante: usando o nome que definimos, sem chaves ou parênteses. Imagine que mais tarde implementaremos comportamento para esse tipo de modo que toda instância de `AlwaysEqual` seja sempre igual a toda instância de qualquer outro tipo, talvez para termos um resultado conhecido em testes. Não precisaríamos de dados para implementar esse comportamento! Você verá no Capítulo 10 como definir traits e implementá-las em qualquer tipo, incluindo unit-like structs.

> ### Ownership dos dados de structs
>
> Na definição da struct `User` da Listagem 5-1, usamos o tipo `String` com ownership em vez do tipo fatia de string `&str`. Essa é uma escolha deliberada porque queremos que cada instância dessa struct seja dona de todos os seus dados e que esses dados permaneçam válidos enquanto a struct inteira for válida.
>
> Também é possível que structs armazenem referências a dados de propriedade de outra parte do programa, mas para isso é necessário usar _lifetimes_, um recurso do Rust que discutiremos no Capítulo 10. Lifetimes garantem que os dados referenciados por uma struct sejam válidos enquanto a struct existir. Digamos que você tente armazenar uma referência em uma struct sem especificar lifetimes, como no código a seguir em `src/main.rs`; isso não funcionará:
>
> **Arquivo: src/main.rs (Este código não compila!)**
>
> ```rust
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> O compilador reclamará de que são necessários especificadores de lifetime:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> No Capítulo 10, discutiremos como corrigir esses erros para que você possa armazenar referências em structs; por enquanto, corrigiremos erros como esses usando tipos com ownership, como `String`, em vez de referências como `&str`.
