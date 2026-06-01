---
title: "Definindo um enum"
chapter_code: 06-01
slug: definindo-um-enum
challenge_day: 7
reading_minutes: 32
---

# Definindo um Enum

Enquanto structs oferecem uma forma de agrupar campos e dados relacionados, como um `Rectangle` com sua `width` e `height`, enums oferecem uma forma de dizer que um valor é um de um conjunto possível de valores. Por exemplo, podemos querer dizer que `Rectangle` é uma das formas possíveis que também incluem `Circle` e `Triangle`. Para isso, o Rust nos permite codificar essas possibilidades como um enum.

Vamos analisar uma situação que talvez queiramos expressar em código e ver por que enums são úteis e mais apropriados que structs neste caso. Digamos que precisamos trabalhar com endereços IP. Atualmente, dois padrões principais são usados para endereços IP: versão quatro e versão seis. Como essas são as únicas possibilidades de endereço IP que nosso programa encontrará, podemos _enumerar_ todas as variantes possíveis, de onde vem o nome _enumeração_.

Qualquer endereço IP pode ser um endereço de versão quatro ou de versão seis, mas não ambos ao mesmo tempo. Essa propriedade dos endereços IP torna a estrutura de dados enum apropriada, porque um valor de enum só pode ser uma de suas variantes. Endereços de versão quatro e versão seis ainda são, em essência, endereços IP, então devem ser tratados como o mesmo tipo quando o código lida com situações que se aplicam a qualquer tipo de endereço IP.

Podemos expressar esse conceito em código definindo uma enumeração `IpAddrKind` e listando os tipos possíveis que um endereço IP pode ser, `V4` e `V6`. Essas são as variantes do enum:

**Arquivo: src/main.rs**

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind` agora é um tipo de dado personalizado que podemos usar em outros lugares do nosso código.

### Valores de enum

Podemos criar instâncias de cada uma das duas variantes de `IpAddrKind` assim:

**Arquivo: src/main.rs**

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

Observe que as variantes do enum ficam no namespace do seu identificador, e usamos dois-pontos duplos para separar os dois. Isso é útil porque agora ambos os valores `IpAddrKind::V4` e `IpAddrKind::V6` são do mesmo tipo: `IpAddrKind`. Podemos então, por exemplo, definir uma função que recebe qualquer `IpAddrKind`:

**Arquivo: src/main.rs**

```rust
fn route(ip_kind: IpAddrKind) {}
```

E podemos chamar essa função com qualquer variante:

```rust
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

Usar enums tem ainda mais vantagens. Pensando mais no nosso tipo de endereço IP, no momento não temos uma forma de armazenar os dados reais do endereço IP; só sabemos de que _tipo_ ele é. Como você acabou de aprender sobre structs no [Capítulo 5](/livro/cap05-00-usando-structs-para-estruturar-dados-relacionados), pode ser tentador resolver esse problema com structs, como mostrado na Listagem 6-1.

**Arquivo: src/main.rs**

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

<a id="listagem-6-1"></a>

[Listagem 6-1](#listagem-6-1): Armazenando os dados e a variante `IpAddrKind` de um endereço IP usando uma `struct`

Aqui, definimos uma struct `IpAddr` que tem dois campos: um campo `kind` do tipo `IpAddrKind` (o enum que definimos antes) e um campo `address` do tipo `String`. Temos duas instâncias dessa struct. A primeira é `home`, e tem o valor `IpAddrKind::V4` como `kind` com dados de endereço associados `127.0.0.1`. A segunda instância é `loopback`. Ela tem a outra variante de `IpAddrKind` como valor de `kind`, `V6`, e tem o endereço `::1` associado. Usamos uma struct para agrupar os valores `kind` e `address`, de modo que agora a variante está associada ao valor.

Porém, representar o mesmo conceito usando apenas um enum é mais conciso: em vez de um enum dentro de uma struct, podemos colocar dados diretamente em cada variante do enum. Esta nova definição do enum `IpAddr` diz que ambas as variantes `V4` e `V6` terão valores `String` associados:

**Arquivo: src/main.rs**

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

Anexamos dados a cada variante do enum diretamente, então não há necessidade de uma struct extra. Aqui, também fica mais fácil ver outro detalhe de como enums funcionam: o nome de cada variante de enum que definimos também se torna uma função que constrói uma instância do enum. Ou seja, `IpAddr::V4()` é uma chamada de função que recebe um argumento `String` e retorna uma instância do tipo `IpAddr`. Obtemos automaticamente essa função construtora definida como resultado de definir o enum.

Há outra vantagem em usar um enum em vez de uma struct: cada variante pode ter tipos e quantidades diferentes de dados associados. Endereços IP de versão quatro sempre terão quatro componentes numéricos com valores entre 0 e 255. Se quiséssemos armazenar endereços `V4` como quatro valores `u8`, mas ainda expressar endereços `V6` como um valor `String`, não conseguiríamos com uma struct. Enums lidam com esse caso com facilidade:

**Arquivo: src/main.rs**

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

Mostramos várias formas diferentes de definir estruturas de dados para armazenar endereços IP de versão quatro e versão seis. Porém, como se vê, querer armazenar endereços IP e codificar de que tipo são é tão comum que a biblioteca padrão tem uma definição que podemos usar! Vejamos como a biblioteca padrão define `IpAddr`. Ela tem exatamente o enum e as variantes que definimos e usamos, mas incorpora os dados de endereço dentro das variantes na forma de duas structs diferentes, definidas de forma distinta para cada variante:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Este código ilustra que você pode colocar qualquer tipo de dado dentro de uma variante de enum: strings, tipos numéricos ou structs, por exemplo. Você pode até incluir outro enum! Além disso, tipos da biblioteca padrão muitas vezes não são muito mais complicados do que o que você poderia criar.

Observe que, embora a biblioteca padrão contenha uma definição para `IpAddr`, ainda podemos criar e usar nossa própria definição sem conflito porque não trouxemos a definição da biblioteca padrão para o nosso escopo. Falaremos mais sobre trazer tipos para o escopo no [Capítulo 7](/livro/cap07-00-gerenciando-projetos-crescentes-com-packages-crates-e-modulos).

Vamos olhar outro exemplo de enum na Listagem 6-2: este tem uma grande variedade de tipos embutidos em suas variantes.

**Arquivo: src/main.rs**

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

<a id="listagem-6-2"></a>

[Listagem 6-2](#listagem-6-2): Um enum `Message` cujas variantes armazenam quantidades e tipos diferentes de valores

Este enum tem quatro variantes com tipos diferentes:

- `Quit`: não tem dados associados de forma alguma
- `Move`: tem campos nomeados, como uma struct
- `Write`: inclui uma única `String`
- `ChangeColor`: inclui três valores `i32`

Definir um enum com variantes como as da Listagem 6-2 é semelhante a definir diferentes tipos de definições de struct, exceto que o enum não usa a palavra-chave `struct` e todas as variantes são agrupadas sob o tipo `Message`. As structs a seguir poderiam armazenar os mesmos dados que as variantes do enum anterior:

**Arquivo: src/main.rs**

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

Mas se usássemos structs diferentes, cada uma com seu próprio tipo, não poderíamos definir tão facilmente uma função que recebesse qualquer um desses tipos de mensagens como poderíamos com o enum `Message` da Listagem 6-2, que é um único tipo.

Há mais uma semelhança entre enums e structs: assim como podemos definir métodos em structs usando `impl`, também podemos definir métodos em enums. Aqui está um método chamado `call` que poderíamos definir no nosso enum `Message`:

**Arquivo: src/main.rs**

```rust
impl Message {
    fn call(&self) {
        // o corpo do método seria definido aqui
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

O corpo do método usaria `self` para obter o valor no qual chamamos o método. Neste exemplo, criamos uma variável `m` com o valor `Message::Write(String::from("hello"))`, e esse será o valor de `self` no corpo do método `call` quando `m.call()` for executado.

Vamos olhar outro enum na biblioteca padrão que é muito comum e útil: `Option`.

### O enum `Option`

Esta seção explora um estudo de caso de `Option`, outro enum definido pela biblioteca padrão. O tipo `Option` codifica o cenário muito comum em que um valor pode ser algo ou nada.

Por exemplo, se você solicitar o primeiro item de uma lista não vazia, obterá um valor. Se solicitar o primeiro item de uma lista vazia, não obterá nada. Expressar esse conceito em termos do sistema de tipos significa que o compilador pode verificar se você tratou todos os casos que deveria tratar; essa funcionalidade pode evitar bugs extremamente comuns em outras linguagens de programação.

O design de linguagens de programação muitas vezes é pensado em termos de quais recursos você inclui, mas os recursos que você exclui também são importantes. O Rust não tem o recurso _null_ que muitas outras linguagens têm. _Null_ é um valor que significa que não há valor ali. Em linguagens com null, variáveis podem estar sempre em um de dois estados: null ou não-null.

Na apresentação de 2009 “Null References: The Billion Dollar Mistake”, Tony Hoare, o inventor do null, disse o seguinte:

> Chamo isso de meu erro de bilhão de dólares. Naquela época, eu estava projetando o primeiro sistema de tipos abrangente para referências em uma linguagem orientada a objetos. Meu objetivo era garantir que todo uso de referências fosse absolutamente seguro, com verificação realizada automaticamente pelo compilador. Mas não resisti à tentação de colocar uma referência null, simplesmente porque era tão fácil de implementar. Isso levou a inúmeros erros, vulnerabilidades e falhas de sistema, que provavelmente causaram um bilhão de dólares em dor e dano nos últimos quarenta anos.

O problema com valores null é que, se você tentar usar um valor null como se não fosse null, obterá algum tipo de erro. Como essa propriedade null ou não-null é onipresente, é extremamente fácil cometer esse tipo de erro.

Porém, o conceito que null tenta expressar ainda é útil: um null é um valor que está inválido ou ausente por algum motivo.

O problema não é realmente o conceito, mas a implementação particular. Assim, o Rust não tem nulls, mas tem um enum que pode codificar o conceito de um valor estar presente ou ausente. Esse enum é `Option<T>`, e é [definido pela biblioteca padrão](https://doc.rust-lang.org/std/option/enum.Option.html) assim:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

O enum `Option<T>` é tão útil que está até incluído no prelude; você não precisa trazê-lo para o escopo explicitamente. Suas variantes também estão no prelude: você pode usar `Some` e `None` diretamente sem o prefixo `Option::`. O enum `Option<T>` ainda é apenas um enum regular, e `Some(T)` e `None` ainda são variantes do tipo `Option<T>`.

A sintaxe `<T>` é um recurso do Rust que ainda não discutimos. É um parâmetro de tipo genérico, e cobriremos generics com mais detalhes no [Capítulo 10](/livro/cap10-00-tipos-genericos-traits-e-lifetimes). Por enquanto, tudo o que você precisa saber é que `<T>` significa que a variante `Some` do enum `Option` pode conter um pedaço de dado de qualquer tipo, e que cada tipo concreto usado no lugar de `T` torna o tipo geral `Option<T>` um tipo diferente. Aqui estão alguns exemplos de uso de valores `Option` para armazenar tipos numéricos e de caractere:

**Arquivo: src/main.rs**

```rust
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;
```

O tipo de `some_number` é `Option<i32>`. O tipo de `some_char` é `Option<char>`, que é um tipo diferente. O Rust pode inferir esses tipos porque especificamos um valor dentro da variante `Some`. Para `absent_number`, o Rust exige que anotemos o tipo `Option` geral: o compilador não pode inferir o tipo que a variante `Some` correspondente conterá olhando apenas para um valor `None`. Aqui, dizemos ao Rust que queremos que `absent_number` seja do tipo `Option<i32>`.

Quando temos um valor `Some`, sabemos que um valor está presente e que o valor está dentro do `Some`. Quando temos um valor `None`, em certo sentido isso significa a mesma coisa que null: não temos um valor válido. Então, por que ter `Option<T>` é melhor que ter null?

Em resumo, porque `Option<T>` e `T` (onde `T` pode ser qualquer tipo) são tipos diferentes, o compilador não nos deixa usar um valor `Option<T>` como se fosse definitivamente um valor válido. Por exemplo, este código não compilará, porque tenta somar um `i8` a um `Option<i8>`:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

Se executarmos este código, recebemos uma mensagem de erro como esta:

```console
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            `&i8` implements `Add<i8>`
            `&i8` implements `Add`
            `i8` implements `Add<&i8>`
            `i8` implements `Add`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` (bin "enums") due to 1 previous error
```

Intenso! Na prática, essa mensagem de erro significa que o Rust não entende como somar um `i8` e um `Option<i8>`, porque são tipos diferentes. Quando temos um valor de um tipo como `i8` em Rust, o compilador garante que sempre temos um valor válido. Podemos prosseguir com confiança sem precisar verificar null antes de usar esse valor. Só quando temos um `Option<i8>` (ou qualquer tipo de valor com que estejamos trabalhando) precisamos nos preocupar com a possibilidade de não ter um valor, e o compilador garantirá que tratemos esse caso antes de usar o valor.

Em outras palavras, você precisa converter um `Option<T>` em um `T` antes de poder realizar operações de `T` com esse valor. Em geral, isso ajuda a capturar um dos problemas mais comuns com null: assumir que algo não é null quando na verdade é.

Eliminar o risco de assumir incorretamente um valor não-null ajuda você a ter mais confiança no seu código. Para ter um valor que pode possivelmente ser null, você deve optar explicitamente tornando o tipo desse valor `Option<T>`. Então, quando usar esse valor, é obrigatório tratar explicitamente o caso em que o valor é null. Em todo lugar onde um valor tem um tipo que não é `Option<T>`, você pode assumir com segurança que o valor não é null. Essa foi uma decisão de design deliberada do Rust para limitar a onipresença de null e aumentar a segurança do código Rust.

Então, como obter o valor `T` de uma variante `Some` quando você tem um valor do tipo `Option<T>` para poder usar esse valor? O enum `Option<T>` tem um grande número de métodos úteis em várias situações; você pode consultá-los na [documentação](https://doc.rust-lang.org/std/option/enum.Option.html). Familiarizar-se com os métodos em `Option<T>` será extremamente útil na sua jornada com Rust.

Em geral, para usar um valor `Option<T>`, você quer código que trate cada variante. Você quer código que execute apenas quando tiver um valor `Some(T)`, e esse código pode usar o `T` interno. Você quer outro código que execute apenas se tiver um valor `None`, e esse código não terá um valor `T` disponível. A expressão `match` é uma construção de fluxo de controle que faz exatamente isso quando usada com enums: executará código diferente dependendo de qual variante do enum tiver, e esse código pode usar os dados dentro do valor correspondente.
