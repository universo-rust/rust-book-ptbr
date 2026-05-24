---
title: "Strings"
chapter_code: 08-02
slug: strings
---

# Armazenando Texto Codificado em UTF-8 com Strings

Falamos sobre strings no Capítulo 4, mas agora as veremos com mais profundidade. Rustaceans iniciantes costumam travar em strings por uma combinação de três motivos: a propensão do Rust para expor erros possíveis, strings serem uma estrutura de dados mais complicada do que muitos programadores creditam, e UTF-8. Esses fatores se combinam de uma forma que pode parecer difícil quando você vem de outras linguagens de programação.

Discutimos strings no contexto de coleções porque strings são implementadas como uma coleção de bytes, mais alguns métodos para fornecer funcionalidade útil quando esses bytes são interpretados como texto. Nesta seção, falaremos sobre as operações em `String` que todo tipo de coleção tem, como criar, atualizar e ler. Também discutiremos as formas em que `String` é diferente das outras coleções, a saber, como indexar em uma `String` é complicado pelas diferenças entre como pessoas e computadores interpretam dados de `String`.

### Definindo strings

Primeiro, definiremos o que queremos dizer com o termo _string_. O Rust tem apenas um tipo string na linguagem principal, que é a fatia de string `str` geralmente vista em sua forma emprestada, `&str`. No Capítulo 4, falamos sobre fatias de string, que são referências a alguns dados de string codificados em UTF-8 armazenados em outro lugar. Literais de string, por exemplo, são armazenados no binário do programa e, portanto, são fatias de string.

O tipo `String`, que é fornecido pela biblioteca padrão do Rust em vez de codificado na linguagem principal, é um tipo string codificado em UTF-8, com ownership, mutável e expansível. Quando rustaceans se referem a “strings” em Rust, podem estar se referindo ao tipo `String` ou ao tipo fatia de string `&str`, não apenas um deles. Embora esta seção trate em grande parte de `String`, ambos os tipos são usados intensamente na biblioteca padrão do Rust, e tanto `String` quanto fatias de string são codificados em UTF-8.

### Criando uma nova string

Muitas das mesmas operações disponíveis com `Vec<T>` também estão disponíveis com `String`, porque `String` é na verdade implementada como um wrapper em torno de um vetor de bytes com algumas garantias, restrições e capacidades extras. Um exemplo de função que funciona da mesma forma com `Vec<T>` e `String` é a função `new` para criar uma instância, mostrada na Listagem 8-11.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::new();
}
```

<a id="listagem-8-11"></a>

[Listagem 8-11](#listagem-8-11): Criando uma nova `String` vazia

Esta linha cria uma nova string vazia chamada `s`, na qual podemos então carregar dados. Frequentemente, teremos alguns dados iniciais com os quais queremos começar a string. Para isso, usamos o método `to_string`, que está disponível em qualquer tipo que implementa a trait `Display`, como literais de string fazem. A Listagem 8-12 mostra dois exemplos.

**Arquivo: src/main.rs**

```rust
fn main() {
    let data = "initial contents";

    let s = data.to_string();

    // O método também funciona em um literal diretamente:
    let s = "initial contents".to_string();
}
```

<a id="listagem-8-12"></a>

[Listagem 8-12](#listagem-8-12): Usando o método `to_string` para criar uma `String` a partir de um literal de string

Este código cria uma string contendo `initial contents`.

Também podemos usar a função `String::from` para criar uma `String` a partir de um literal de string. O código da Listagem 8-13 é equivalente ao da Listagem 8-12 que usa `to_string`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s = String::from("initial contents");
}
```

<a id="listagem-8-13"></a>

[Listagem 8-13](#listagem-8-13): Usando a função `String::from` para criar uma `String` a partir de um literal de string

Como strings são usadas para tantas coisas, podemos usar muitas APIs genéricas diferentes para strings, fornecendo-nos muitas opções. Algumas podem parecer redundantes, mas todas têm seu lugar! Neste caso, `String::from` e `to_string` fazem a mesma coisa, então qual você escolhe é uma questão de estilo e legibilidade.

Lembre-se de que strings são codificadas em UTF-8, então podemos incluir quaisquer dados devidamente codificados nelas, como mostrado na Listagem 8-14.

**Arquivo: src/main.rs**

```rust
fn main() {
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
}
```

<a id="listagem-8-14"></a>

[Listagem 8-14](#listagem-8-14): Armazenando saudações em diferentes idiomas em strings

Todos estes são valores `String` válidos.

### Atualizando uma string

Uma `String` pode crescer em tamanho e seu conteúdo pode mudar, assim como o conteúdo de um `Vec<T>`, se você empurrar mais dados nela. Além disso, você pode convenientemente usar o operador `+` ou a macro `format!` para concatenar valores `String`.

#### Acrescentando com `push_str` ou `push`

Podemos fazer uma `String` crescer usando o método `push_str` para acrescentar uma fatia de string, como mostrado na Listagem 8-15.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar");
}
```

<a id="listagem-8-15"></a>

[Listagem 8-15](#listagem-8-15): Acrescentando uma fatia de string a uma `String` usando o método `push_str`

Depois dessas duas linhas, `s` conterá `foobar`. O método `push_str` recebe uma fatia de string porque não queremos necessariamente tomar ownership do parâmetro. Por exemplo, no código da Listagem 8-16, queremos poder usar `s2` depois de acrescentar seu conteúdo a `s1`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
}
```

<a id="listagem-8-16"></a>

[Listagem 8-16](#listagem-8-16): Usando uma fatia de string depois de acrescentar seu conteúdo a uma `String`

Se o método `push_str` tomasse ownership de `s2`, não poderíamos imprimir seu valor na última linha. Porém, este código funciona como esperaríamos!

O método `push` recebe um único caractere como parâmetro e o adiciona à `String`. A Listagem 8-17 adiciona a letra _l_ a uma `String` usando o método `push`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::from("lo");
    s.push('l');
}
```

<a id="listagem-8-17"></a>

[Listagem 8-17](#listagem-8-17): Adicionando um caractere a um valor `String` usando `push`

Como resultado, `s` conterá `lol`.

#### Concatenando com `+` ou `format!`

Muitas vezes, você quer combinar duas strings existentes. Uma forma de fazer isso é usar o operador `+`, como mostrado na Listagem 8-18.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note que s1 foi movida aqui e não pode mais ser usada
}
```

<a id="listagem-8-18"></a>

[Listagem 8-18](#listagem-8-18): Usando o operador `+` para combinar dois valores `String` em um novo valor `String`

A string `s3` conterá `Hello, world!`. O motivo de `s1` não ser mais válida depois da adição, e o motivo de termos usado uma referência a `s2`, tem a ver com a assinatura do método que é chamado quando usamos o operador `+`. O operador `+` usa o método `add`, cuja assinatura se parece com isto:

```rust
fn add(self, s: &str) -> String {
```

Na biblioteca padrão, você verá `add` definido usando generics e tipos associados. Aqui, substituímos por tipos concretos, que é o que acontece quando chamamos este método com valores `String`. Discutiremos generics no Capítulo 10. Esta assinatura nos dá as pistas necessárias para entender os aspectos complicados do operador `+`.

Primeiro, `s2` tem um `&`, o que significa que estamos adicionando uma referência da segunda string à primeira string. Isso ocorre por causa do parâmetro `s` na função `add`: só podemos adicionar uma fatia de string a uma `String`; não podemos adicionar dois valores `String` juntos. Mas espere — o tipo de `&s2` é `&String`, não `&str`, como especificado no segundo parâmetro de `add`. Então, por que a Listagem 8-18 compila?

O motivo de podermos usar `&s2` na chamada a `add` é que o compilador pode fazer coerção de `&String` para `&str`. Quando chamamos o método `add`, o Rust usa uma coerção de dereferência, que aqui transforma `&s2` em `&s2[..]`. Discutiremos coerção de dereferência com mais profundidade no Capítulo 15. Como `add` não toma ownership do parâmetro `s`, `s2` ainda será uma `String` válida depois desta operação.

Segundo, podemos ver na assinatura que `add` toma ownership de `self` porque `self` _não_ tem um `&`. Isso significa que `s1` na Listagem 8-18 será movida para a chamada a `add` e não será mais válida depois disso. Então, embora `let s3 = s1 + &s2;` pareça que copiará ambas as strings e criará uma nova, esta declaração na verdade toma ownership de `s1`, acrescenta uma cópia do conteúdo de `s2` e então devolve ownership do resultado. Em outras palavras, parece que está fazendo muitas cópias, mas não está; a implementação é mais eficiente que copiar.

Se precisarmos concatenar várias strings, o comportamento do operador `+` fica incômodo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
}
```

Neste ponto, `s` será `tic-tac-toe`. Com todos os `+` e `"` caracteres, é difícil ver o que está acontecendo. Para combinar strings de formas mais complicadas, podemos usar a macro `format!`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
}
```

Este código também define `s` como `tic-tac-toe`. A macro `format!` funciona como `println!`, mas em vez de imprimir a saída na tela, retorna uma `String` com o conteúdo. A versão do código usando `format!` é muito mais fácil de ler, e o código gerado pela macro `format!` usa referências para que esta chamada não tome ownership de nenhum de seus parâmetros.

### Indexando em strings

Em muitas outras linguagens de programação, acessar caracteres individuais em uma string referenciando-os por índice é uma operação válida e comum. Porém, se você tentar acessar partes de uma `String` usando sintaxe de indexação em Rust, obterá um erro. Considere o código inválido da Listagem 8-19.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let s1 = String::from("hi");
    let h = s1[0];
}
```

<a id="listagem-8-19"></a>

[Listagem 8-19](#listagem-8-19): Tentando usar sintaxe de indexação com uma `String`

Este código resultará no seguinte erro:

```bash
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

O erro conta a história: strings Rust não suportam indexação. Mas por quê? Para responder a essa pergunta, precisamos discutir como o Rust armazena strings na memória.

#### Representação interna

Uma `String` é um wrapper em torno de um `Vec<u8>`. Vamos olhar algumas de nossas strings de exemplo UTF-8 devidamente codificadas da Listagem 8-14. Primeiro, esta:

```rust
let hello = String::from("Hola");
```

Neste caso, `hello.len()` será `4`, o que significa que o vetor que armazena a string `"Hola"` tem 4 bytes de comprimento. Cada uma dessas letras ocupa 1 byte quando codificada em UTF-8. A linha a seguir, porém, pode surpreendê-lo (note que esta string começa com a letra cirílica maiúscula _Ze_, não o número 3):

```rust
let hello = String::from("Здравствуйте");
```

Se lhe perguntassem qual é o comprimento da string, você poderia dizer 12. Na verdade, a resposta do Rust é 24: esse é o número de bytes necessários para codificar “Здравствуйте” em UTF-8, porque cada valor escalar Unicode nessa string ocupa 2 bytes de armazenamento. Portanto, um índice nos bytes da string nem sempre corresponderá a um valor escalar Unicode válido. Para demonstrar, considere este código Rust inválido:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

Você já sabe que `answer` não será `З`, a primeira letra. Quando codificado em UTF-8, o primeiro byte de `З` é `208` e o segundo é `151`, então pareceria que `answer` deveria ser `208`, mas `208` não é um caractere válido por si só. Retornar `208` provavelmente não é o que um usuário quereria se pedisse a primeira letra desta string; porém, esses são os únicos dados que o Rust tem no índice de byte 0. Usuários geralmente não querem o valor do byte retornado, mesmo que a string contenha apenas letras latinas: se `&"hi"[0]` fosse código válido que retornasse o valor do byte, retornaria `104`, não `h`.

A resposta, então, é que para evitar retornar um valor inesperado e causar bugs que podem não ser descobertos imediatamente, o Rust não compila este código e previne mal-entendidos cedo no processo de desenvolvimento.

#### Bytes, valores escalares e clusters de grafemas

Outro ponto sobre UTF-8 é que há na verdade três formas relevantes de olhar strings da perspectiva do Rust: como bytes, valores escalares e clusters de grafemas (a coisa mais próxima do que chamaríamos de _letras_).

Se olharmos a palavra hindi “नमस्ते” escrita no script Devanagari, ela é armazenada como um vetor de valores `u8` que se parece com isto:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

São 18 bytes e é como os computadores armazenam esses dados. Se os olharmos como valores escalares Unicode, que é o que o tipo `char` do Rust é, esses bytes se parecem com isto:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Há seis valores `char` aqui, mas o quarto e o sexto não são letras: são diacríticos que não fazem sentido sozinhos. Por fim, se os olharmos como clusters de grafemas, obteríamos o que uma pessoa chamaria das quatro letras que compõem a palavra hindi:

```text
["न", "म", "स्", "ते"]
```

O Rust fornece diferentes formas de interpretar os dados brutos de string que os computadores armazenam para que cada programa possa escolher a interpretação de que precisa, independentemente do idioma humano dos dados.

Um motivo final pelo qual o Rust não nos permite indexar em uma `String` para obter um caractere é que operações de indexação esperam sempre levar tempo constante (O(1)). Mas não é possível garantir esse desempenho com uma `String`, porque o Rust teria que percorrer o conteúdo desde o início até o índice para determinar quantos caracteres válidos havia.

### Fatiando strings

Indexar em uma string muitas vezes é uma má ideia porque não está claro qual deveria ser o tipo de retorno da operação de indexação de string: um valor de byte, um caractere, um cluster de grafemas ou uma fatia de string. Se você realmente precisar usar índices para criar fatias de string, portanto, o Rust pede que seja mais específico.

Em vez de indexar usando `[]` com um único número, você pode usar `[]` com um intervalo para criar uma fatia de string contendo bytes particulares:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aqui, `s` será um `&str` que contém os primeiros 4 bytes da string. Mencionamos antes que cada um desses caracteres tinha 2 bytes, o que significa que `s` será `Зд`.

Se tentássemos fatiar apenas parte dos bytes de um caractere com algo como `&hello[0..1]`, o Rust entraria em pânico em tempo de execução da mesma forma que se um índice inválido fosse acessado em um vetor:

```bash
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`

thread 'main' panicked at src/main.rs:4:19:
byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Você deve ter cautela ao criar fatias de string com intervalos, porque isso pode fazer seu programa entrar em pânico.

### Iterando sobre strings

A melhor forma de operar em pedaços de strings é ser explícito sobre se quer caracteres ou bytes. Para valores escalares Unicode individuais, use o método `chars`. Chamar `chars` em “Зд” separa e retorna dois valores do tipo `char`, e você pode iterar sobre o resultado para acessar cada elemento:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Este código imprimirá o seguinte:

```text
З
д
```

Alternativamente, o método `bytes` retorna cada byte bruto, o que pode ser apropriado para seu domínio:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Este código imprimirá os 4 bytes que compõem esta string:

```text
208
151
208
180
```

Mas lembre-se de que valores escalares Unicode válidos podem ser compostos por mais de 1 byte.

Obter clusters de grafemas de strings, como com o script Devanagari, é complexo, então essa funcionalidade não é fornecida pela biblioteca padrão. Crates estão disponíveis em [crates.io](https://crates.io/) se essa for a funcionalidade de que você precisa.

### Lidando com as complexidades de strings

Para resumir, strings são complicadas. Diferentes linguagens de programação fazem escolhas diferentes sobre como apresentar essa complexidade ao programador. O Rust escolheu tornar o tratamento correto de dados `String` o comportamento padrão para todos os programas Rust, o que significa que programadores precisam pensar mais em lidar com dados UTF-8 desde o início. Essa troca expõe mais da complexidade de strings do que é aparente em outras linguagens de programação, mas evita que você precise lidar com erros envolvendo caracteres não ASCII mais tarde no seu ciclo de vida de desenvolvimento.

A boa notícia é que a biblioteca padrão oferece muita funcionalidade construída sobre os tipos `String` e `&str` para ajudar a lidar com essas situações complexas corretamente. Certifique-se de consultar a documentação para métodos úteis como `contains` para buscar em uma string e `replace` para substituir partes de uma string por outra string.

Vamos mudar para algo um pouco menos complexo: hash maps!
