---
title: "Strings"
chapter_code: 08-02
slug: strings
---

# Armazenando Texto Codificado em UTF-8 com Strings

Falamos sobre strings no Capítulo 4; agora vamos mergulhar mais fundo. Rustaceans iniciantes costumam travar em strings por três motivos combinados: a tendência do Rust a tornar visíveis erros potenciais, strings serem uma estrutura de dados mais complexa do que muitos programadores imaginam, e UTF-8. Juntos, esses fatores podem parecer intimidantes quando você vem de outras linguagens.

Abordamos strings no contexto de coleções porque elas são implementadas como uma coleção de bytes, com métodos extras para trabalhar com esse conteúdo como texto. Nesta seção, veremos as operações comuns a todo tipo de coleção — criar, atualizar e ler — aplicadas a `String`. Também veremos o que torna `String` diferente das demais coleções: indexar uma `String` é complicado porque pessoas e computadores interpretam esses dados de formas distintas.

### Definindo strings

Primeiro, vamos definir o que queremos dizer com _string_. O Rust tem apenas um tipo string no núcleo da linguagem: a fatia de string `str`, geralmente usada na forma emprestada `&str`. No Capítulo 4, vimos fatias de string — referências a dados de texto codificados em UTF-8 armazenados em outro lugar. Literais de string, por exemplo, ficam no binário do programa e, portanto, são fatias de string.

O tipo `String`, fornecido pela biblioteca padrão (e não embutido no núcleo da linguagem), é uma string UTF-8 expansível, mutável e com ownership. Quando rustaceans falam em “strings” em Rust, podem estar se referindo tanto a `String` quanto a `&str` — não apenas a um dos dois. Embora esta seção foque em `String`, ambos os tipos aparecem o tempo todo na biblioteca padrão, e tanto `String` quanto fatias de string usam codificação UTF-8.

### Criando uma nova string

Muitas operações disponíveis em `Vec<T>` também funcionam com `String`, porque `String` é, na prática, um wrapper em torno de um vetor de bytes, com garantias, restrições e capacidades extras. A função `new`, usada para criar uma instância, é um exemplo que se comporta igual nos dois tipos — veja a Listagem 8-11.

**Arquivo: src/main.rs**

```rust
let mut s = String::new();
```

<a id="listagem-8-11"></a>

[Listagem 8-11](#listagem-8-11): Criando uma nova `String` vazia

Essa linha cria uma string vazia chamada `s`, pronta para receber dados. Muitas vezes, já temos um conteúdo inicial. Nesse caso, usamos o método `to_string`, disponível em qualquer tipo que implementa a trait `Display` — literais de string incluídos. A Listagem 8-12 mostra dois exemplos.

**Arquivo: src/main.rs**

```rust
let data = "initial contents";

let s = data.to_string();

// The method also works on a literal directly:
let s = "initial contents".to_string();
```

<a id="listagem-8-12"></a>

[Listagem 8-12](#listagem-8-12): Usando o método `to_string` para criar uma `String` a partir de um literal de string

Esse código cria uma string com o conteúdo `initial contents`.

Também podemos usar `String::from` para criar uma `String` a partir de um literal. O código da Listagem 8-13 é equivalente ao da Listagem 8-12.

**Arquivo: src/main.rs**

```rust
let s = String::from("initial contents");
```

<a id="listagem-8-13"></a>

[Listagem 8-13](#listagem-8-13): Usando a função `String::from` para criar uma `String` a partir de um literal de string

Como strings aparecem em tantos contextos, a biblioteca oferece várias APIs genéricas — algumas parecem redundantes, mas cada uma tem seu lugar. Aqui, `String::from` e `to_string` fazem a mesma coisa; a escolha é questão de estilo e legibilidade.

Lembre-se: strings são UTF-8, então podemos armazenar qualquer texto corretamente codificado, como na Listagem 8-14.

**Arquivo: src/main.rs**

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שלום");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

<a id="listagem-8-14"></a>

[Listagem 8-14](#listagem-8-14): Armazenando saudações em diferentes idiomas em strings

Todos esses são valores `String` válidos.

### Atualizando uma string

Uma `String` pode crescer e mudar de conteúdo — como um `Vec<T>` — quando você adiciona mais dados. Também dá para concatenar valores `String` com o operador `+` ou com a macro `format!`.

#### Acrescentando com `push_str` ou `push`

Podemos fazer uma `String` crescer com o método `push_str`, que acrescenta uma fatia de string, como na Listagem 8-15.

**Arquivo: src/main.rs**

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

<a id="listagem-8-15"></a>

[Listagem 8-15](#listagem-8-15): Acrescentando uma fatia de string a uma `String` com o método `push_str`

Depois dessas duas linhas, `s` contém `foobar`. O `push_str` recebe uma fatia de string porque nem sempre queremos tomar ownership do parâmetro. Na Listagem 8-16, por exemplo, precisamos continuar usando `s2` depois de acrescentar seu conteúdo a `s1`.

**Arquivo: src/main.rs**

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {s2}");
```

<a id="listagem-8-16"></a>

[Listagem 8-16](#listagem-8-16): Usando uma fatia de string depois de acrescentar seu conteúdo a uma `String`

Se `push_str` tomasse ownership de `s2`, não poderíamos imprimi-lo na última linha. Felizmente, o código se comporta como esperado.

O método `push` recebe um único caractere e o adiciona à `String`. A Listagem 8-17 acrescenta a letra _l_ com `push`.

**Arquivo: src/main.rs**

```rust
let mut s = String::from("lo");
s.push('l');
```

<a id="listagem-8-17"></a>

[Listagem 8-17](#listagem-8-17): Adicionando um caractere a uma `String` com `push`

No fim, `s` contém `lol`.

#### Concatenando com `+` ou `format!`

Muitas vezes você quer juntar duas strings existentes. Uma forma é usar o operador `+`, como na Listagem 8-18.

**Arquivo: src/main.rs**

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

<a id="listagem-8-18"></a>

[Listagem 8-18](#listagem-8-18): Usando o operador `+` para combinar duas `String` em uma nova `String`

A string `s3` contém `Hello, world!`. Por que `s1` deixa de ser válida depois da adição? E por que usamos referência em `s2`? A resposta está na assinatura do método chamado por `+`. O operador usa o método `add`, cuja assinatura se parece com isto:

```rust
fn add(self, s: &str) -> String {
```

Na biblioteca padrão, `add` é definido com generics e tipos associados. Aqui usamos tipos concretos — o que acontece quando chamamos o método com `String`. Veremos generics no Capítulo 10. Essa assinatura já explica os detalhes mais confusos do operador `+`.

Primeiro: `s2` leva um `&`, ou seja, estamos somando uma referência da segunda string à primeira. Isso vem do parâmetro `s` em `add`: só dá para somar uma fatia de string a uma `String`, não duas `String` de uma vez. Mas espere — o tipo de `&s2` é `&String`, não `&str`, como pede o segundo parâmetro de `add`. Por que a Listagem 8-18 compila?

Porque o compilador pode converter `&String` em `&str`. Na chamada a `add`, o Rust aplica coerção de dereferência e transforma `&s2` em `&s2[..]`. Veremos coerção de dereferência com mais profundidade no Capítulo 15. Como `add` não toma ownership de `s`, `s2` continua válida depois da operação.

Segundo: na assinatura, `add` toma ownership de `self` — note que `self` _não_ tem `&`. Isso significa que `s1` na Listagem 8-18 é movida para a chamada a `add` e deixa de ser válida. Então, embora `let s3 = s1 + &s2;` pareça copiar as duas strings e criar uma nova, na verdade a expressão toma ownership de `s1`, acrescenta uma cópia do conteúdo de `s2` e devolve ownership do resultado. Parece que há muitas cópias, mas não há — a implementação é mais eficiente do que copiar tudo.

Para concatenar várias strings, o operador `+` fica incômodo:

**Arquivo: src/main.rs**

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

Aqui, `s` fica `tic-tac-toe`. Com tantos `+` e aspas, fica difícil enxergar o que está acontecendo. Para combinações mais elaboradas, use a macro `format!`:

**Arquivo: src/main.rs**

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
```

Esse código também define `s` como `tic-tac-toe`. A macro `format!` funciona como `println!`, mas em vez de imprimir na tela, retorna uma `String`. A versão com `format!` é bem mais legível — e o código gerado usa referências, então a chamada não toma ownership de nenhum parâmetro.

### Indexando em strings

Em muitas linguagens, acessar caracteres individuais por índice é comum e válido. Em Rust, porém, usar sintaxe de indexação em uma `String` gera erro. Veja o código inválido da Listagem 8-19.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let s1 = String::from("hi");
let h = s1[0];
```

<a id="listagem-8-19"></a>

[Listagem 8-19](#listagem-8-19): Tentando usar sintaxe de indexação com uma `String`

Esse código produz o seguinte erro:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `str` cannot be indexed by `{integer}`
 --> src/main.rs:3:16
  |
3 |     let h = s1[0];
  |                ^ string indices are ranges of `usize`
  |
  = help: the trait `SliceIndex<str>` is not implemented for `{integer}`
  = note: you can use `.chars().nth()` or `.bytes().nth()`
          for more information, see chapter 8 in The Book: <https://doc.rust-lang.org/book/ch08-02-strings.html#indexing-into-strings>
  = help: the following other types implement trait `SliceIndex<T>`:
            `usize` implements `SliceIndex<ByteStr>`
            `usize` implements `SliceIndex<[T]>`
  = note: required for `String` to implement `Index<{integer}>`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` (bin "collections") due to 1 previous error
```

O próprio erro deixa claro: strings em Rust não suportam indexação. Mas por quê? Para responder, precisamos ver como o Rust armazena strings na memória.

#### Representação interna

Uma `String` é um wrapper em torno de um `Vec<u8>`. Vamos olhar alguns exemplos UTF-8 da Listagem 8-14. Primeiro:

```rust
let hello = String::from("Hola");
```

Aqui, `hello.len()` retorna `4` — o vetor que guarda `"Hola"` tem 4 bytes. Cada letra ocupa 1 byte em UTF-8. A linha seguinte, porém, pode surpreender (note que a string começa com a letra cirílica maiúscula _Ze_, não com o número 3):

```rust
let hello = String::from("Здравствуйте");
```

Se alguém perguntasse o comprimento da string, você poderia dizer 12. O Rust, porém, responde 24: são os bytes necessários para codificar “Здравствуйте” em UTF-8, porque cada valor escalar Unicode ocupa 2 bytes. Portanto, um índice nos bytes da string nem sempre corresponde a um valor escalar Unicode válido. Para ver isso na prática, considere este código inválido:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

Você já sabe que `answer` não será `З`, a primeira letra. Em UTF-8, o primeiro byte de `З` é `208` e o segundo é `151` — então `answer` pareceria ser `208`, mas `208` sozinho não é um caractere válido. Devolver `208` provavelmente não é o que o usuário espera ao pedir a primeira letra; ainda assim, é tudo o que o Rust tem no índice de byte 0. Na prática, ninguém quer o valor bruto do byte — nem em strings só com letras latinas: se `&"hi"[0]` fosse válido e retornasse o byte, viria `104`, não `h`.

Para evitar valores inesperados e bugs difíceis de detectar, o Rust simplesmente não compila esse código — e elimina o mal-entendido cedo no desenvolvimento.

#### Bytes, valores escalares e clusters de grafemas

Há três formas relevantes de olhar strings em Rust: como bytes, como valores escalares e como clusters de grafemas (o mais próximo do que chamamos de _letras_).

A palavra hindi “नमस्ते”, escrita em Devanagari, é armazenada como um vetor de `u8` assim:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

São 18 bytes — é assim que o computador guarda os dados. Como valores escalares Unicode (o que o tipo `char` representa), esses bytes ficam assim:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Há seis valores `char`, mas o quarto e o sexto não são letras: são diacríticos que não fazem sentido sozinhos. Como clusters de grafemas, veríamos as quatro “letras” que um falante de hindi reconheceria na palavra:

```text
["न", "म", "स्", "ते"]
```

O Rust oferece formas distintas de interpretar os dados brutos de uma string para que cada programa escolha a visão de que precisa — independentemente do idioma.

Outro motivo pelo qual o Rust não permite indexar uma `String` para obter um caractere: indexação deveria ser O(1). Com `String`, isso não é garantido — o Rust precisaria percorrer o conteúdo desde o início até o índice para contar quantos caracteres válidos existem.

### Fatiando strings

Indexar uma string costuma ser má ideia porque não fica claro o que a operação deveria retornar: um byte, um caractere, um cluster de grafemas ou uma fatia de string. Se você realmente precisar de índices para criar fatias, o Rust exige que seja mais específico.

Em vez de `[]` com um único número, use `[]` com um intervalo para obter uma fatia de string com bytes específicos:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aqui, `s` é um `&str` com os primeiros 4 bytes da string. Como cada caractere dessa palavra ocupa 2 bytes, `s` corresponde a `Зд`.

Se tentarmos fatiar só parte dos bytes de um caractere — por exemplo, `&hello[0..1]` — o Rust entra em pânico em tempo de execução, como acontece com índice inválido em um vetor:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`

thread 'main' panicked at src/main.rs:4:19:
byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Tenha cuidado ao criar fatias de string com intervalos — isso pode derrubar o programa.

### Iterando sobre strings

A melhor forma de trabalhar com pedaços de string é deixar explícito se você quer caracteres ou bytes. Para valores escalares Unicode individuais, use `chars`. Chamar `chars` em “Зд” separa e retorna dois valores `char`; dá para iterar sobre o resultado:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Esse código imprime:

```text
З
д
```

Alternativamente, `bytes` retorna cada byte bruto — útil em alguns domínios:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Esse código imprime os 4 bytes que compõem a string:

```text
208
151
208
180
```

Lembre-se: um valor escalar Unicode válido pode ocupar mais de 1 byte.

Obter clusters de grafemas — como no Devanagari — é complexo; a biblioteca padrão não oferece isso. Há crates em [crates.io](https://crates.io/) se você precisar dessa funcionalidade.

### Lidando com as complexidades de strings

Resumindo: strings são complicadas. Cada linguagem decide como expor essa complexidade ao programador. O Rust optou por tornar o tratamento correto de `String` o comportamento padrão — o que exige pensar em UTF-8 desde cedo. Essa escolha deixa a complexidade mais visível do que em outras linguagens, mas evita erros com caracteres não ASCII mais adiante no desenvolvimento.

A boa notícia é que a biblioteca padrão oferece muita funcionalidade em cima de `String` e `&str` para lidar com esses casos. Vale consultar a documentação: métodos como `contains` (busca) e `replace` (substituição de trechos) ajudam bastante.

Vamos para algo um pouco menos complexo: hash maps!
