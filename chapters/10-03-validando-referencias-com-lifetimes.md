---
title: "Validando referências com lifetimes"
chapter_code: 10-03
slug: validando-referencias-com-lifetimes
---

# Validando referências com lifetimes

Lifetimes são outro tipo de generic que já estamos usando. Em vez de garantir que um tipo tenha o comportamento que queremos, lifetimes garantem que referências sejam válidas enquanto precisarmos delas.

Um detalhe que não discutimos na seção Referências e Borrowing do Capítulo 4 é que toda referência em Rust tem um _lifetime_, que é o escopo em que essa referência é válida. Na maior parte do tempo, lifetimes são implícitos e inferidos, assim como na maior parte do tempo os tipos são inferidos. Só somos obrigados a anotar tipos quando vários tipos são possíveis. De forma semelhante, devemos anotar lifetimes quando os lifetimes das referências podem se relacionar de algumas formas diferentes. O Rust exige que anotemos os relacionamentos usando parâmetros de lifetime genéricos para garantir que as referências usadas em tempo de execução serão definitivamente válidas.

Anotar lifetimes nem é um conceito que a maioria das outras linguagens de programação tem, então isso vai parecer desconhecido. Embora não cubramos lifetimes por completo neste capítulo, discutiremos formas comuns em que você pode encontrar sintaxe de lifetime para que se familiarize com o conceito.

### Referências penduradas

O principal objetivo dos lifetimes é impedir referências penduradas, que, se fossem permitidas, fariam um programa referenciar dados diferentes dos que pretende referenciar. Considere o programa da Listagem 10-16, que tem um escopo externo e um interno.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {r}");
}
```

<a id="listagem-10-16"></a>

[Listagem 10-16](#listagem-10-16): Tentativa de usar uma referência cujo valor saiu de escopo

> Nota: Os exemplos nas Listagens 10-16, 10-17 e 10-23 declaram variáveis sem dar um valor inicial, de modo que o nome da variável existe no escopo externo. À primeira vista, isso pode parecer conflitar com o Rust não ter valores nulos. Porém, se tentarmos usar uma variável antes de dar um valor a ela, obteremos um erro em tempo de compilação, o que mostra que de fato o Rust não permite valores nulos.

O escopo externo declara uma variável chamada `r` sem valor inicial, e o escopo interno declara uma variável chamada `x` com valor inicial `5`. Dentro do escopo interno, tentamos definir o valor de `r` como referência a `x`. Depois, o escopo interno termina e tentamos imprimir o valor em `r`. Este código não compilará, porque o valor ao qual `r` se refere saiu de escopo antes de tentarmos usá-lo. Esta é a mensagem de erro:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
5 |         let x = 5;
  |             - binding `x` declared here
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {r}");
  |                   - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

A mensagem de erro diz que a variável `x` “não vive o suficiente”. O motivo é que `x` estará fora de escopo quando o escopo interno terminar na linha 7. Mas `r` ainda é válida para o escopo externo; como seu escopo é maior, dizemos que ela “vive mais”. Se o Rust permitisse que este código funcionasse, `r` estaria referenciando memória que foi desalocada quando `x` saiu de escopo, e qualquer coisa que tentássemos fazer com `r` não funcionaria corretamente. Então, como o Rust determina que este código é inválido? Usa um borrow checker.

### O borrow checker

O compilador Rust tem um _borrow checker_ que compara escopos para determinar se todos os empréstimos são válidos. A Listagem 10-17 mostra o mesmo código da Listagem 10-16, mas com anotações mostrando os lifetimes das variáveis.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {r}");   //          |
}                         // ---------+
```

<a id="listagem-10-17"></a>

[Listagem 10-17](#listagem-10-17): Anotações dos lifetimes de `r` e `x`, nomeados `'a` e `'b`, respectivamente

Aqui, anotamos o lifetime de `r` com `'a` e o lifetime de `x` com `'b`. Como você pode ver, o bloco interno `'b` é muito menor que o bloco de lifetime externo `'a`. Em tempo de compilação, o Rust compara o tamanho dos dois lifetimes e vê que `r` tem lifetime `'a`, mas se refere a memória com lifetime `'b`. O programa é rejeitado porque `'b` é mais curto que `'a`: o assunto da referência não vive tanto quanto a referência.

A Listagem 10-18 corrige o código para que não tenha referência pendurada e compile sem erros.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {r}");   //   |       |
                          // --+       |
}                         // ----------+
```

<a id="listagem-10-18"></a>

[Listagem 10-18](#listagem-10-18): Uma referência válida porque os dados têm lifetime maior que a referência

Aqui, `x` tem o lifetime `'b`, que neste caso é maior que `'a`. Isso significa que `r` pode referenciar `x` porque o Rust sabe que a referência em `r` sempre será válida enquanto `x` for válido.

Agora que você sabe onde estão os lifetimes das referências e como o Rust analisa lifetimes para garantir que referências serão sempre válidas, vamos explorar lifetimes genéricos em parâmetros e valores de retorno de funções.

### Lifetimes genéricos em funções

Escreveremos uma função que retorna o slice de string mais longo entre dois. Esta função receberá dois slices de string e retornará um único slice de string. Depois de implementarmos a função `longest`, o código da Listagem 10-19 deve imprimir `A string mais longa é abcd`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("A string mais longa é {result}");
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

<a id="listagem-10-19"></a>

[Listagem 10-19](#listagem-10-19): Uma função `main` que chama a função `longest` para encontrar o maior de dois slices de string

Observe que queremos que a função receba slices de string, que são referências, em vez de `String`, porque não queremos que `longest` tome ownership de seus parâmetros. Consulte a seção [String slices como parâmetros](/livro/cap04-03-slices#string-slices-como-parâmetros) do Capítulo 4 para mais discussão sobre por que os parâmetros que usamos na Listagem 10-19 são os que queremos.

Se tentarmos implementar a função `longest` como mostrado na Listagem 10-20, ela não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("A string mais longa é {result}");
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

<a id="listagem-10-20"></a>

[Listagem 10-20](#listagem-10-20): Uma implementação da função `longest` que retorna o maior de dois slices de string, mas ainda não compila

Em vez disso, obtemos o seguinte erro que fala sobre lifetimes:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

O texto de ajuda revela que o tipo de retorno precisa de um parâmetro de lifetime genérico, porque o Rust não pode dizer se a referência retornada se refere a `x` ou a `y`. Na verdade, nós também não sabemos, porque o bloco `if` no corpo desta função retorna referência a `x` e o bloco `else` retorna referência a `y`!

Ao definir esta função, não sabemos os valores concretos que serão passados a ela, então não sabemos se o caso `if` ou o `else` será executado. Também não sabemos os lifetimes concretos das referências que serão passadas, então não podemos olhar os escopos como nas Listagens 10-17 e 10-18 para determinar se a referência que retornamos sempre será válida. O borrow checker também não consegue determinar isso, porque não sabe como os lifetimes de `x` e `y` se relacionam com o lifetime do valor de retorno. Para corrigir este erro, adicionaremos parâmetros de lifetime genéricos que definem o relacionamento entre as referências para que o borrow checker possa fazer sua análise.

### Sintaxe de anotação de lifetime

Anotações de lifetime não mudam quanto tempo qualquer uma das referências vive. Em vez disso, descrevem os relacionamentos dos lifetimes de várias referências entre si sem afetar os lifetimes. Assim como funções podem aceitar qualquer tipo quando a assinatura especifica um parâmetro de tipo genérico, funções podem aceitar referências com qualquer lifetime especificando um parâmetro de lifetime genérico.

Anotações de lifetime têm sintaxe um pouco incomum: os nomes dos parâmetros de lifetime devem começar com apóstrofo (`'`) e costumam ser minúsculos e bem curtos, como tipos genéricos. A maioria das pessoas usa o nome `'a` para a primeira anotação de lifetime. Colocamos anotações de parâmetro de lifetime após o `&` de uma referência, com espaço separando a anotação do tipo da referência.

Aqui estão alguns exemplos — uma referência a um `i32` sem parâmetro de lifetime, uma referência a um `i32` que tem parâmetro de lifetime chamado `'a`, e uma referência mutável a um `i32` que também tem o lifetime `'a`:

```rust
&i32        // uma referência
&'a i32     // uma referência com lifetime explícito
&'a mut i32 // uma referência mutável com lifetime explícito
```

Uma anotação de lifetime sozinha não tem muito significado, porque as anotações servem para dizer ao Rust como parâmetros de lifetime genéricos de várias referências se relacionam entre si. Vamos examinar como as anotações de lifetime se relacionam no contexto da função `longest`.

### Em assinaturas de função

Para usar anotações de lifetime em assinaturas de função, precisamos declarar os parâmetros de lifetime genéricos dentro de colchetes angulares entre o nome da função e a lista de parâmetros, como fizemos com parâmetros de tipo genérico.

Queremos que a assinatura expresse a seguinte restrição: a referência retornada será válida enquanto ambos os parâmetros forem válidos. Este é o relacionamento entre os lifetimes dos parâmetros e o valor de retorno. Nomearemos o lifetime `'a` e então o adicionaremos a cada referência, como mostrado na Listagem 10-21.

**Arquivo: src/main.rs**

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("A string mais longa é {result}");
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

<a id="listagem-10-21"></a>

[Listagem 10-21](#listagem-10-21): Definição da função `longest` especificando que todas as referências na assinatura devem ter o mesmo lifetime `'a`

Este código deve compilar e produzir o resultado que queremos quando usado com a função `main` da Listagem 10-19.

A assinatura da função agora diz ao Rust que, para algum lifetime `'a`, a função recebe dois parâmetros, ambos slices de string que vivem pelo menos tanto quanto o lifetime `'a`. A assinatura também diz ao Rust que o slice de string retornado pela função viverá pelo menos tanto quanto o lifetime `'a`. Na prática, isso significa que o lifetime da referência retornada por `longest` é o mesmo que o menor dos lifetimes dos valores referenciados pelos argumentos da função. Esses relacionamentos são o que queremos que o Rust use ao analisar este código.

Lembre-se: quando especificamos os parâmetros de lifetime nesta assinatura de função, não estamos mudando os lifetimes de valores passados ou retornados. Em vez disso, estamos especificando que o borrow checker deve rejeitar valores que não obedeçam a essas restrições. Observe que a função `longest` não precisa saber exatamente quanto tempo `x` e `y` viverão, apenas que algum escopo pode ser substituído por `'a` que satisfará esta assinatura.

Ao anotar lifetimes em funções, as anotações vão na assinatura da função, não no corpo. As anotações de lifetime tornam-se parte do contrato da função, como os tipos na assinatura. Ter assinaturas de função que contêm o contrato de lifetime significa que a análise que o compilador Rust faz pode ser mais simples. Se houver problema na forma como uma função está anotada ou na forma como é chamada, os erros do compilador podem apontar para a parte do código e as restrições com mais precisão. Se, em vez disso, o compilador Rust fizesse mais inferências sobre os relacionamentos de lifetimes que pretendíamos, talvez só pudesse apontar para um uso do código muitos passos depois da causa do problema.

Quando passamos referências concretas a `longest`, o lifetime concreto substituído por `'a` é a parte do escopo de `x` que se sobrepõe ao escopo de `y`. Em outras palavras, o lifetime genérico `'a` receberá o lifetime concreto igual ao menor dos lifetimes de `x` e `y`. Como anotamos a referência retornada com o mesmo parâmetro de lifetime `'a`, a referência retornada também será válida pelo comprimento do menor dos lifetimes de `x` e `y`.

Vejamos como as anotações de lifetime restringem a função `longest` passando referências que têm lifetimes concretos diferentes. A Listagem 10-22 é um exemplo direto.

**Arquivo: src/main.rs**

```rust
fn main() {
    let string1 = String::from("A string longa é longa");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("A string mais longa é: {result}");
    }
}

```

<a id="listagem-10-22"></a>

[Listagem 10-22](#listagem-10-22): Usando a função `longest` com referências a valores `String` que têm lifetimes concretos diferentes

Neste exemplo, `string1` é válida até o fim do escopo externo, `string2` é válida até o fim do escopo interno e `result` referencia algo válido até o fim do escopo interno. Execute este código e verá que o borrow checker aprova; compilará e imprimirá `A string longa é: A string longa é longa`.

Em seguida, vamos tentar um exemplo que mostra que o lifetime da referência em `result` deve ser o menor lifetime dos dois argumentos. Moveremos a declaração da variável `result` para fora do escopo interno, mas deixaremos a atribuição do valor a `result` dentro do escopo com `string2`. Depois, moveremos o `println!` que usa `result` para fora do escopo interno, após o escopo interno ter terminado. O código da Listagem 10-23 não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let string1 = String::from("string longa é longa");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("A string mais longa é {result}");
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

<a id="listagem-10-23"></a>

[Listagem 10-23](#listagem-10-23): Tentativa de usar `result` depois que `string2` saiu de escopo

Quando tentarmos compilar este código, obteremos este erro:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
5 |         let string2 = String::from("xyz");
  |             ------- binding `string2` declared here
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("A string mais longa é {result}");
  |                                      ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

O erro mostra que, para `result` ser válido na instrução `println!`, `string2` precisaria ser válida até o fim do escopo externo. O Rust sabe disso porque anotamos os lifetimes dos parâmetros da função e do valor de retorno usando o mesmo parâmetro de lifetime `'a`.

Como humanos, podemos olhar este código e ver que `string1` é mais longa que `string2` e, portanto, `result` conterá referência a `string1`. Como `string1` ainda não saiu de escopo, uma referência a `string1` ainda será válida para a instrução `println!`. Porém, o compilador não pode ver que a referência é válida neste caso. Dissemos ao Rust que o lifetime da referência retornada por `longest` é o mesmo que o menor dos lifetimes das referências passadas. Portanto, o borrow checker não permite o código da Listagem 10-23 por possivelmente ter referência inválida.

Tente criar mais experimentos que variem os valores e lifetimes das referências passadas à função `longest` e como a referência retornada é usada. Formule hipóteses sobre se seus experimentos passarão no borrow checker antes de compilar; depois, verifique se acertou!

### Relacionamentos

A forma como você precisa especificar parâmetros de lifetime depende do que sua função faz. Por exemplo, se mudássemos a implementação de `longest` para sempre retornar o primeiro parâmetro em vez do slice de string mais longo, não precisaríamos especificar lifetime no parâmetro `y`. O seguinte código compilará:

**Arquivo: src/main.rs**

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "efghijklmnopqrstuvwxyz";

    let result = longest(string1.as_str(), string2);
    println!("A string mais longa é {result}");
}

fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

Especificamos um parâmetro de lifetime `'a` para o parâmetro `x` e o tipo de retorno, mas não para o parâmetro `y`, porque o lifetime de `y` não tem relacionamento com o lifetime de `x` ou com o valor de retorno.

Ao retornar uma referência de uma função, o parâmetro de lifetime do tipo de retorno precisa corresponder ao parâmetro de lifetime de um dos parâmetros. Se a referência retornada _não_ se refere a um dos parâmetros, deve referir-se a um valor criado dentro desta função. Porém, isso seria uma referência pendurada, porque o valor sairá de escopo no fim da função. Considere esta tentativa de implementação de `longest` que não compilará:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("A string mais longa é {result}");
}

fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("string bem longa");
    result.as_str()
}
```

Mesmo tendo especificado um parâmetro de lifetime `'a` para o tipo de retorno, esta implementação falhará ao compilar porque o lifetime do valor retornado não está relacionado ao lifetime dos parâmetros. Esta é a mensagem de erro que obtemos:

```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return value referencing local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ------^^^^^^^^^
   |     |
   |     returns a value referencing data owned by the current function
   |     `result` is borrowed here

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

O problema é que `result` sai de escopo e é limpo no fim da função `longest`. Também estamos tentando retornar uma referência a `result` da função. Não há forma de especificar parâmetros de lifetime que mudem a referência pendurada, e o Rust não nos deixará criar uma referência pendurada. Neste caso, a melhor correção seria retornar um tipo de dado com ownership em vez de uma referência, para que a função chamadora fique responsável por limpar o valor.

Em última análise, a sintaxe de lifetime trata de conectar os lifetimes de vários parâmetros e valores de retorno de funções. Uma vez conectados, o Rust tem informação suficiente para permitir operações seguras em memória e proibir operações que criariam ponteiros pendurados ou violariam a segurança de memória de outra forma.

### Em definições de struct

Até agora, as structs que definimos armazenam tipos com ownership. Podemos definir structs para armazenar referências, mas nesse caso precisaríamos adicionar uma anotação de lifetime em cada referência na definição da struct. A Listagem 10-24 tem uma struct chamada `ImportantExcerpt` que armazena um slice de string.

**Arquivo: src/main.rs**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Chamo-me Ishmael. Há alguns anos...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

<a id="listagem-10-24"></a>

[Listagem 10-24](#listagem-10-24): Uma struct que armazena uma referência, exigindo anotação de lifetime

Esta struct tem o único campo `part` que armazena um slice de string, que é uma referência. Como com tipos de dados genéricos, declaramos o nome do parâmetro de lifetime genérico dentro de colchetes angulares após o nome da struct para podermos usar o parâmetro de lifetime no corpo da definição da struct. Esta anotação significa que uma instância de `ImportantExcerpt` não pode viver mais que a referência que armazena no campo `part`.

A função `main` aqui cria uma instância da struct `ImportantExcerpt` que armazena referência à primeira frase da `String` de propriedade da variável `novel`. Os dados em `novel` existem antes da instância de `ImportantExcerpt` ser criada. Além disso, `novel` não sai de escopo até depois de `ImportantExcerpt` sair de escopo, então a referência na instância de `ImportantExcerpt` é válida.

### Elisão de lifetimes

Você aprendeu que toda referência tem um lifetime e que precisa especificar parâmetros de lifetime para funções ou structs que usam referências. Porém, tínhamos uma função na Listagem 4-9, mostrada novamente na Listagem 10-25, que compilou sem anotações de lifetime.

**Arquivo: src/main.rs**

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let my_string = String::from("olá mundo");

    // first_word funciona com slices de `String`
    let word = first_word(&my_string[..]);

    let my_string_literal = "olá mundo";

    // first_word funciona com slices de literais de string
    let word = first_word(&my_string_literal[..]);

    // Como literais de string *já são* slices de string,
    // isto também funciona, sem a sintaxe de slice!
    let word = first_word(my_string_literal);
}
```

<a id="listagem-10-25"></a>

[Listagem 10-25](#listagem-10-25): Uma função que definimos na Listagem 4-9 e que compilou sem anotações de lifetime, embora o parâmetro e o tipo de retorno sejam referências

O motivo de esta função compilar sem anotações de lifetime é histórico: em versões iniciais (pré-1.0) do Rust, este código não teria compilado, porque toda referência precisava de lifetime explícito. Naquela época, a assinatura da função teria sido escrita assim:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Depois de escrever muito código Rust, a equipe do Rust descobriu que programadores Rust inseriam as mesmas anotações de lifetime repetidamente em situações particulares. Essas situações eram previsíveis e seguiam alguns padrões determinísticos. Os desenvolvedores programaram esses padrões no código do compilador para que o borrow checker pudesse inferir lifetimes nessas situações e não precisasse de anotações explícitas.

Este pedaço da história do Rust é relevante porque é possível que mais padrões determinísticos surjam e sejam adicionados ao compilador. No futuro, talvez ainda menos anotações de lifetime sejam necessárias.

Os padrões programados na análise de referências do Rust são chamados de _regras de elisão de lifetimes_. Estas não são regras para programadores seguirem; são um conjunto de casos particulares que o compilador considerará, e se seu código se encaixar nesses casos, você não precisa escrever lifetimes explicitamente.

As regras de elisão não fornecem inferência completa. Se ainda houver ambiguidade sobre quais lifetimes as referências têm depois que o Rust aplica as regras, o compilador não adivinhará qual deveria ser o lifetime das referências restantes. Em vez de adivinhar, o compilador dará um erro que você pode resolver adicionando as anotações de lifetime.

Lifetimes em parâmetros de função ou método são chamados de _lifetimes de entrada_, e lifetimes em valores de retorno são chamados de _lifetimes de saída_.

O compilador usa três regras para descobrir os lifetimes das referências quando não há anotações explícitas. A primeira regra se aplica a lifetimes de entrada, e a segunda e a terceira a lifetimes de saída. Se o compilador chegar ao fim das três regras e ainda houver referências para as quais não consegue descobrir lifetimes, o compilador para com um erro. Essas regras se aplicam a definições `fn` e a blocos `impl`.

A primeira regra é que o compilador atribui um parâmetro de lifetime a cada parâmetro que é referência. Em outras palavras, uma função com um parâmetro recebe um parâmetro de lifetime: `fn foo<'a>(x: &'a i32)`; uma função com dois parâmetros recebe dois parâmetros de lifetime separados: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; e assim por diante.

A segunda regra é que, se há exatamente um parâmetro de lifetime de entrada, esse lifetime é atribuído a todos os parâmetros de lifetime de saída: `fn foo<'a>(x: &'a i32) -> &'a i32`.

A terceira regra é que, se há vários parâmetros de lifetime de entrada, mas um deles é `&self` ou `&mut self` porque é um método, o lifetime de `self` é atribuído a todos os parâmetros de lifetime de saída. Esta terceira regra torna métodos muito mais agradáveis de ler e escrever porque são necessários menos símbolos.

Vamos fingir que somos o compilador. Aplicaremos essas regras para descobrir os lifetimes das referências na assinatura da função `first_word` da Listagem 10-25. A assinatura começa sem lifetimes associados às referências:

```rust
fn first_word(s: &str) -> &str {
```

Depois, o compilador aplica a primeira regra, que especifica que cada parâmetro recebe seu próprio lifetime. Chamaremos de `'a` como de costume, então agora a assinatura é esta:

```rust
fn first_word<'a>(s: &'a str) -> &str {
```

A segunda regra se aplica porque há exatamente um lifetime de entrada. A segunda regra especifica que o lifetime do único parâmetro de entrada é atribuído ao lifetime de saída, então a assinatura agora é esta:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Agora todas as referências nesta assinatura de função têm lifetimes, e o compilador pode continuar sua análise sem precisar que o programador anote lifetimes nesta assinatura.

Vejamos outro exemplo, desta vez usando a função `longest` que não tinha parâmetros de lifetime quando começamos a trabalhar com ela na Listagem 10-20:

```rust
fn longest(x: &str, y: &str) -> &str {
```

Vamos aplicar a primeira regra: cada parâmetro recebe seu próprio lifetime. Desta vez temos dois parâmetros em vez de um, então temos dois lifetimes:

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Você pode ver que a segunda regra não se aplica, porque há mais de um lifetime de entrada. A terceira regra também não se aplica, porque `longest` é uma função e não um método, então nenhum dos parâmetros é `self`. Depois de passar pelas três regras, ainda não descobrimos qual é o lifetime do tipo de retorno. É por isso que obtivemos um erro ao tentar compilar o código da Listagem 10-20: o compilador passou pelas regras de elisão de lifetimes, mas ainda não conseguiu descobrir todos os lifetimes das referências na assinatura.

Como a terceira regra realmente só se aplica em assinaturas de método, veremos lifetimes nesse contexto em seguida para entender por que a terceira regra significa que raramente precisamos anotar lifetimes em assinaturas de método.

### Em definições de método

Ao implementar métodos em uma struct com lifetimes, usamos a mesma sintaxe dos parâmetros de tipo genérico, como mostrado na Listagem 10-11. Onde declaramos e usamos os parâmetros de lifetime depende de estarem relacionados aos campos da struct ou aos parâmetros e valores de retorno do método.

Nomes de lifetime para campos de struct sempre precisam ser declarados após a palavra-chave `impl` e então usados após o nome da struct, porque esses lifetimes fazem parte do tipo da struct.

Em assinaturas de método dentro do bloco `impl`, referências podem estar ligadas ao lifetime de referências nos campos da struct, ou podem ser independentes. Além disso, as regras de elisão de lifetimes frequentemente fazem com que anotações de lifetime não sejam necessárias em assinaturas de método. Vejamos alguns exemplos usando a struct chamada `ImportantExcerpt` que definimos na Listagem 10-24.

Primeiro, usaremos um método chamado `level` cujo único parâmetro é referência a `self` e cujo valor de retorno é um `i32`, que não é referência a nada:

**Arquivo: src/main.rs**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}

fn main() {
    let novel = String::from("Chamo-me Ishmael. Há alguns anos...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

A declaração do parâmetro de lifetime após `impl` e seu uso após o nome do tipo são obrigatórios, mas por causa da primeira regra de elisão, não somos obrigados a anotar o lifetime da referência a `self`.

Aqui está um exemplo em que a terceira regra de elisão de lifetime se aplica:

**Arquivo: src/main.rs**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Atenção, por favor: {announcement}");
        self.part
    }
}

fn main() {
    let novel = String::from("Chamo-me Ishmael. Há alguns anos...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

Há dois lifetimes de entrada, então o Rust aplica a primeira regra de elisão e dá a `&self` e `announcement` lifetimes próprios. Depois, como um dos parâmetros é `&self`, o tipo de retorno recebe o lifetime de `&self`, e todos os lifetimes foram contabilizados.

### O lifetime `'static`

Um lifetime especial que precisamos discutir é `'static`, que indica que a referência afetada _pode_ viver durante toda a duração do programa. Todos os literais de string têm o lifetime `'static`, que podemos anotar assim:

```rust
let s: &'static str = "Tenho um lifetime estático.";
```

O texto desta string é armazenado diretamente no binário do programa, que está sempre disponível. Portanto, o lifetime de todos os literais de string é `'static`.

Você pode ver sugestões em mensagens de erro para usar o lifetime `'static`. Mas antes de especificar `'static` como lifetime de uma referência, pense se a referência que você tem realmente vive durante todo o lifetime do seu programa e se quer que isso aconteça. Na maior parte do tempo, uma mensagem de erro sugerindo o lifetime `'static resulta de tentar criar uma referência pendurada ou de incompatibilidade dos lifetimes disponíveis. Nesses casos, a solução é corrigir esses problemas, não especificar o lifetime `'static`.

## Parâmetros de tipo genérico, trait bounds e lifetimes juntos

Vejamos brevemente a sintaxe de especificar parâmetros de tipo genérico, trait bounds e lifetimes em uma única função!

**Arquivo: src/main.rs**

```rust
use std::fmt::Display;

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest_with_an_announcement(
        string1.as_str(),
        string2,
        "Hoje é aniversário de alguém!",
    );
    println!("A string mais longa é {result}");
}

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Anúncio! {ann}");
    if x.len() > y.len() { x } else { y }
}
```

Esta é a função `longest` da Listagem 10-21 que retorna o maior de dois slices de string. Mas agora tem um parâmetro extra chamado `ann` do tipo genérico `T`, que pode ser preenchido por qualquer tipo que implemente a trait `Display`, conforme especificado pela cláusula `where`. Este parâmetro extra será impresso com `{}`, por isso o trait bound `Display` é necessário. Como lifetimes são um tipo de generic, as declarações do parâmetro de lifetime `'a` e do parâmetro de tipo genérico `T` vão na mesma lista dentro dos colchetes angulares após o nome da função.

## Resumo

Cobrimos muito neste capítulo! Agora que você conhece parâmetros de tipo genérico, traits e trait bounds, e parâmetros de lifetime genéricos, está pronto para escrever código sem repetição que funciona em muitas situações diferentes. Parâmetros de tipo genérico permitem aplicar o código a tipos diferentes. Traits e trait bounds garantem que, mesmo que os tipos sejam genéricos, terão o comportamento que o código precisa. Você aprendeu a usar anotações de lifetime para garantir que este código flexível não terá referências penduradas. E toda essa análise acontece em tempo de compilação, o que não afeta o desempenho em tempo de execução!

Acredite ou não, há muito mais a aprender nos tópicos que discutimos neste capítulo: o Capítulo 18 discute trait objects, que são outra forma de usar traits. Há também cenários mais complexos envolvendo anotações de lifetime que você só precisará em cenários muito avançados; para esses, leia a [Referência do Rust](https://doc.rust-lang.org/reference/trait-bounds.html). Mas a seguir você aprenderá a escrever testes em Rust para garantir que seu código funciona como deveria.
