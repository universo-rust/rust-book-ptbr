---
title: "O tipo slice"
chapter_code: 04-03
slug: slices
---

# O tipo slice

_Slices_ permitem que você faça referência a uma sequência contígua de elementos em uma coleção. Um slice é um tipo de referência e, portanto, não tem ownership.

Aqui está um pequeno problema de programação: escreva uma função que receba uma string de palavras separadas por espaços e retorne a primeira palavra que encontrar nessa string. Se a função não encontrar um espaço na string, a string inteira deve ser uma palavra, então a string inteira deve ser retornada.

> **Nota:** Para fins de introdução aos slices, assumimos apenas ASCII nesta seção; uma discussão mais completa sobre tratamento de UTF-8 está na seção Armazenando texto codificado em UTF-8 com Strings do Capítulo 8.

Vamos ver como escreveríamos a assinatura dessa função sem usar slices, para entender o problema que os slices vão resolver:

```rust,ignore
fn first_word(s: &String) -> ?
```

A função `first_word` tem um parâmetro do tipo `&String`. Não precisamos de ownership, então isso está bem. (Em Rust idiomático, funções não tomam ownership de seus argumentos a menos que precisem, e os motivos ficarão claros conforme avançamos.) Mas o que devemos retornar? Não temos realmente uma forma de falar sobre _parte_ de uma string. Porém, poderíamos retornar o índice do fim da palavra, indicado por um espaço. Vamos tentar isso, como mostra a Listagem 4-7.

**Arquivo: src/main.rs**

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {}
```

<a id="listagem-4-7"></a>

[Listagem 4-7](#listagem-4-7): A função `first_word` que retorna um valor de índice de byte no parâmetro `String`

Como precisamos percorrer a `String` elemento por elemento e verificar se um valor é um espaço, converteremos nossa `String` em um array de bytes usando o método `as_bytes`:

```rust,ignore
let bytes = s.as_bytes();
```

Em seguida, criamos um iterador sobre o array de bytes usando o método `iter`:

```rust,ignore
for (i, &item) in bytes.iter().enumerate() {
```

Discutiremos iteradores com mais detalhes no Capítulo 13. Por enquanto, saiba que `iter` é um método que retorna cada elemento em uma coleção e que `enumerate` envolve o resultado de `iter` e retorna cada elemento como parte de uma tupla. O primeiro elemento da tupla retornada por `enumerate` é o índice, e o segundo é uma referência ao elemento. Isso é um pouco mais conveniente do que calcular o índice nós mesmos.

Como o método `enumerate` retorna uma tupla, podemos usar padrões para desestruturar essa tupla. Discutiremos padrões mais no Capítulo 6. No loop `for`, especificamos um padrão que tem `i` para o índice na tupla e `&item` para o byte único na tupla. Como obtemos uma referência ao elemento de `.iter().enumerate()`, usamos `&` no padrão.

Dentro do loop `for`, procuramos o byte que representa o espaço usando a sintaxe de literal de byte. Se encontrarmos um espaço, retornamos a posição. Caso contrário, retornamos o comprimento da string usando `s.len()`.

```rust,ignore
        if item == b' ' {
            return i;
        }
    }

    s.len()
```

Agora temos uma forma de descobrir o índice do fim da primeira palavra na string, mas há um problema. Estamos retornando um `usize` sozinho, mas ele só é um número significativo no contexto da `&String`. Em outras palavras, por ser um valor separado da `String`, não há garantia de que continuará válido no futuro. Considere o programa da Listagem 4-8, que usa a função `first_word` da Listagem 4-7.

**Arquivo: src/main.rs**

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word receberá o valor 5

    s.clear(); // isso esvazia a String, deixando-a igual a ""

    // word ainda tem o valor 5 aqui, mas s não tem mais conteúdo que
    // pudéssemos usar de forma significativa com o valor 5 — word está inválido!
}
```

<a id="listagem-4-8"></a>

[Listagem 4-8](#listagem-4-8): Armazenando o resultado da chamada a `first_word` e depois alterando o conteúdo da `String`

Este programa compila sem erros e também compilaria se usássemos `word` depois de chamar `s.clear()`. Como `word` não está conectado ao estado de `s`, `word` ainda contém o valor `5`. Poderíamos usar esse valor `5` com a variável `s` para tentar extrair a primeira palavra, mas isso seria um bug, porque o conteúdo de `s` mudou desde que salvamos `5` em `word`.

Ter que se preocupar com o índice em `word` ficando dessincronizado com os dados em `s` é tedioso e propenso a erros! Gerenciar esses índices é ainda mais frágil se escrevermos uma função `second_word`. Sua assinatura teria que ser assim:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Agora estamos rastreando um índice inicial _e_ um final, e temos ainda mais valores calculados a partir de dados em um estado particular, mas que não estão ligados a esse estado. Temos três variáveis não relacionadas circulando que precisam ser mantidas em sincronia.

Felizmente, Rust tem uma solução para esse problema: _string slices_.

## String slices

Um _string slice_ é uma referência a uma sequência contígua dos elementos de uma `String`, e se parece com isto:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
}
```

Em vez de uma referência à `String` inteira, `hello` é uma referência a uma porção da `String`, especificada no trecho extra `[0..5]`. Criamos slices usando um intervalo entre colchetes, especificando `[índice_inicial..índice_final]`, em que _`índice_inicial`_ é a primeira posição no slice e _`índice_final`_ é um a mais que a última posição no slice. Internamente, a estrutura de dados do slice armazena a posição inicial e o comprimento do slice, que corresponde a _`índice_final`_ menos _`índice_inicial`_. Assim, no caso de `let world = &s[6..11];`, `world` seria um slice que contém um ponteiro para o byte no índice 6 de `s` com um valor de comprimento 5.

*Figura 4-7: Um string slice referenciando parte de uma `String`. Três tabelas: uma tabela representa os dados na stack de s, que aponta para o byte no índice 0 em uma tabela dos dados da string "hello world" na heap. A terceira tabela representa os dados na stack do slice world, que tem um valor de comprimento 5 e aponta para o byte 6 da tabela de dados na heap.*

Com a sintaxe de intervalo `..` do Rust, se você quiser começar no índice 0, pode omitir o valor antes dos dois pontos. Em outras palavras, estes são iguais:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Da mesma forma, se o slice incluir o último byte da `String`, você pode omitir o número final. Isso significa que estes são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Você também pode omitir ambos os valores para obter um slice da string inteira. Assim, estes são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> **Nota:** Índices de intervalo de string slice devem ocorrer em limites válidos de caracteres UTF-8. Se você tentar criar um string slice no meio de um caractere multibyte, seu programa encerrará com um erro.

Com todas essas informações em mente, vamos reescrever `first_word` para retornar um slice. O tipo que significa "string slice" é escrito como `&str`:

**Arquivo: src/main.rs**

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {}
```

Obtemos o índice do fim da palavra da mesma forma que na Listagem 4-7, procurando a primeira ocorrência de um espaço. Quando encontramos um espaço, retornamos um string slice usando o início da string e o índice do espaço como índices inicial e final.

Agora, quando chamamos `first_word`, recebemos de volta um único valor ligado aos dados subjacentes. O valor é composto por uma referência ao ponto inicial do slice e pelo número de elementos no slice.

Retornar um slice também funcionaria para uma função `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Agora temos uma API direta que é muito mais difícil de usar incorretamente, porque o compilador garantirá que as referências à `String` permaneçam válidas. Lembre-se do bug no programa da Listagem 4-8, quando obtivemos o índice do fim da primeira palavra, mas depois limpamos a string e nosso índice ficou inválido? Esse código era logicamente incorreto, mas não mostrava erros imediatos. Os problemas apareceriam depois se continuássemos tentando usar o índice da primeira palavra com uma string esvaziada. Slices tornam esse bug impossível e nos avisam muito antes de que há um problema no código. Usar a versão com slice de `first_word` gera um erro em tempo de compilação:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // erro!

    println!("the first word is: {word}");
}
```

Veja o erro do compilador:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:18:5
  |
16 |     let word = first_word(&s);
  |                       ------- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
  |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {word}");
  |                                  ---- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Lembre-se das regras de borrowing: se temos uma referência imutável a algo, não podemos também obter uma referência mutável. Como `clear` precisa truncar a `String`, precisa obter uma referência mutável. O `println!` depois da chamada a `clear` usa a referência em `word`, então a referência imutável ainda precisa estar ativa nesse ponto. Rust impede que a referência mutável em `clear` e a referência imutável em `word` existam ao mesmo tempo, e a compilação falha. Não só o Rust tornou nossa API mais fácil de usar, como também eliminou uma classe inteira de erros em tempo de compilação!

### Literais de string como slices

Lembre-se de que falamos sobre literais de string serem armazenados dentro do binário. Agora que sabemos sobre slices, podemos entender corretamente os literais de string:

```rust
let s = "Hello, world!";
```

O tipo de `s` aqui é `&str`: é um slice apontando para aquele ponto específico do binário. É por isso que literais de string são imutáveis; `&str` é uma referência imutável.

### String slices como parâmetros

Saber que você pode obter slices de literais e valores `String` nos leva a mais uma melhoria em `first_word`: sua assinatura:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Um Rustacean mais experiente escreveria a assinatura mostrada na Listagem 4-9, porque ela permite usar a mesma função tanto em valores `&String` quanto em valores `&str`.

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
```

<a id="listagem-4-9"></a>

[Listagem 4-9](#listagem-4-9): Melhorando a função `first_word` usando um string slice como tipo do parâmetro `s`

Se temos um string slice, podemos passá-lo diretamente. Se temos uma `String`, podemos passar um slice da `String` ou uma referência à `String`. Essa flexibilidade aproveita _deref coercions_, um recurso que veremos na seção Usando deref coercions em funções e métodos do Capítulo 15.

Definir uma função para receber um string slice em vez de uma referência a `String` torna nossa API mais geral e útil sem perder funcionalidade:

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
    let my_string = String::from("hello world");

    // `first_word` funciona em slices de `String`, parciais ou completos.
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` também funciona em referências a `String`s, equivalentes
    // a slices completos de `String`.
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` funciona em slices de literais de string, parciais ou
    // completos.
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Como literais de string *já são* string slices,
    // isto também funciona, sem a sintaxe de slice!
    let word = first_word(my_string_literal);
}
```

## Outros slices

String slices, como você pode imaginar, são específicos para strings. Mas há um tipo de slice mais geral também. Considere este array:

```rust
let a = [1, 2, 3, 4, 5];
```

Assim como podemos querer referenciar parte de uma string, podemos querer referenciar parte de um array. Fazemos assim:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Este slice tem o tipo `&[i32]`. Funciona da mesma forma que string slices, armazenando uma referência ao primeiro elemento e um comprimento. Você usará esse tipo de slice para várias outras coleções. Discutiremos essas coleções em detalhes quando falarmos de vetores no Capítulo 8.

## Resumo

Os conceitos de ownership, borrowing e slices garantem segurança de memória em programas Rust em tempo de compilação. A linguagem Rust lhe dá controle sobre o uso de memória da mesma forma que outras linguagens de programação de sistemas. Mas ter o dono dos dados limpando automaticamente esses dados quando sai de escopo significa que você não precisa escrever e depurar código extra para obter esse controle.

Ownership afeta como muitas outras partes do Rust funcionam, então falaremos desses conceitos novamente ao longo do restante do livro. Vamos seguir para o Capítulo 5 e ver como agrupar pedaços de dados em uma `struct`.
