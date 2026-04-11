---
title: "Tipos de dados"
chapter_code: 03-02
slug: tipos-de-dados
---

# Tipos de dados

Todo valor em Rust possui um tipo de dado específico, que informa ao Rust que tipo de informação está sendo usada para que ele saiba como trabalhar com esses dados. Vamos analisar dois subconjuntos de tipos de dados: escalares e compostos.

Tenha em mente que Rust é uma linguagem estaticamente tipada, o que significa que ela precisa conhecer os tipos de todas as variáveis em tempo de compilação. O compilador geralmente consegue inferir qual tipo queremos usar com base no valor atribuído e na forma como esse valor é utilizado. Em casos em que vários tipos são possíveis — como quando convertemos uma `String` para um tipo numérico usando `parse`, como vimos na seção [Comparando o palpite com o número secreto](https://universorust.com.br/area-membro/livro-rust/2-programando-um-jogo-de-adivinhacao#comparando-o-palpite-com-o-numero-secreto) no Capítulo 2 — precisamos adicionar uma anotação de tipo, como no exemplo a seguir:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Se não adicionarmos a anotação `: u32`, o Rust exibirá um erro informando que o compilador precisa de mais informações para determinar o tipo correto.

```bash
$ cargo build
   Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations)
error[E0284]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^        ----- type must be known at this point
  |
  = note: cannot satisfy `<_ as FromStr>::Err == _`
help: consider giving `guess` an explicit type
  |
2 |     let guess: /* Type */ = "42".parse().expect("Not a number!");
  |              ++++++++++++

For more information about this error, try `rustc --explain E0284`.
error: could not compile `no_type_annotations` (bin "no_type_annotations") due to 1 previous error
```

Você verá anotações de tipo diferentes ao trabalhar com outros tipos de dados.

## Tipos escalares

Um tipo *escalar* representa um único valor. O Rust possui quatro tipos escalares principais: inteiros, números de ponto flutuante, booleanos e caracteres. Você provavelmente já reconhece esses tipos de outras linguagens de programação. Vamos ver como eles funcionam no Rust.

## Tipos inteiros

Um inteiro é um número que não possui parte fracionária. No Capítulo 2, usamos um tipo inteiro: o `u32`. Essa declaração de tipo indica que o valor associado a ele deve ser um inteiro sem sinal (os tipos inteiros com sinal começam com `i` em vez de `u`) e que ocupa 32 bits de espaço. A Tabela 3-1 mostra os tipos inteiros nativos do Rust. Podemos usar qualquer uma dessas variações para declarar o tipo de um valor inteiro.

A tabela abaixo representa os tipos inteiros nativos do Rust:

| Tamanho | Signed (com sinal) | Unsigned (sem sinal) |
|---|---|---|
| 8 bits | `i8` | `u8` |
| 16 bits | `i16` | `u16` |
| 32 bits | `i32` | `u32` |
| 64 bits | `i64` | `u64` |
| 128 bits | `i128` | `u128` |
| Dependente da arquitetura | `isize` | `usize` |

[Tabela 3-1](#tabela-3-1): Tipos de inteiros em Rust

Cada variação pode ser com sinal ou sem sinal e possui um tamanho definido. Com sinal *(signed)* e sem sinal *(unsigned)* indicam se o número pode ser negativo — ou seja, se ele precisa ter um sinal *(signed)* ou se será sempre positivo e, portanto, pode ser representado sem sinal *(unsigned)*. É como escrever números no papel: quando o sinal importa, usamos o + ou o −; quando é seguro assumir que o número é positivo, escrevemos sem sinal. Números com sinal são armazenados usando a representação de complemento de dois.

Cada variação com sinal pode armazenar valores de **−(2ⁿ⁻¹)** até **2ⁿ⁻¹ − 1**, inclusive, onde *n* é o número de bits usados por aquela variação. Assim, um `i8` pode armazenar valores de **−128** a **127**. Já as variações sem sinal podem armazenar valores de **0** até **2ⁿ − 1**; portanto, um `u8` pode armazenar valores de **0** a **255**.

Além disso, os tipos `isize` e `usize` dependem da arquitetura do computador em que o programa está sendo executado: 64 bits em arquiteturas de 64 bits e 32 bits em arquiteturas de 32 bits.

Você pode escrever literais inteiros em qualquer um dos formatos mostrados na Tabela 3-2. Note que literais numéricos que podem representar vários tipos permitem um sufixo de tipo, como `57u8`, para indicar explicitamente o tipo. Os literais também podem usar `_` como separador visual para facilitar a leitura, como em `1_000`, que tem o mesmo valor que `1000`.

| Tipo de literal | Exemplo |
|---|---|
| Decimal | `98_222` |
| Hexadecimal | `0xff` |
| Octal | `0o77` |
| Binário | `0b1111_0000` |
| Byte (`u8` apenas) | `b'A'` |

[Tabela 3-2](#tabela-3-2): Literais de números inteiros em Rust

Então, como saber qual tipo de inteiro usar? Se você não tiver certeza, os valores padrão do Rust geralmente são um bom ponto de partida: os tipos inteiros usam `i32` por padrão. A principal situação em que você vai usar `isize` ou `usize` é quando estiver lidando com índices de algum tipo de coleção (como vetores ou arrays).

> #### Estouro de inteiros (integer overflow)
>
> Imagine que você tenha uma variável do tipo `u8`, que pode armazenar valores entre 0 e 255. Se você tentar atribuir a essa variável um valor fora desse intervalo, como 256, ocorre o chamado estouro de inteiro (*integer overflow*).
>
> Quando o programa é compilado em modo debug, o Rust inclui verificações de estouro de inteiros. Se um estouro acontecer, o programa entra em pânico em tempo de execução. Em Rust, o termo *panic* significa que o programa é encerrado com um erro. Esse comportamento será explicado com mais detalhes no Capítulo 9.
>
> Já quando o programa é compilado em modo release, usando a flag `--release`, o Rust não faz essas verificações. Nesse caso, se ocorrer um estouro, o Rust aplica o comportamento chamado wrap-around, baseado em complemento de dois (*two's complement*).
>
> Em termos simples, valores que ultrapassam o máximo permitido pelo tipo "voltam" para o menor valor possível. No caso de um `u8`, o valor 256 se torna 0, o valor 257 se torna 1, e assim por diante.
>
> O programa não entra em pânico, mas a variável passa a ter um valor que provavelmente não é o que você esperava. Por isso, confiar nesse comportamento automático de estouro é considerado um erro de lógica.
>
> Para lidar explicitamente com a possibilidade de estouro, o Rust oferece, na biblioteca padrão, alguns conjuntos de métodos para tipos numéricos primitivos:
>
> - `wrapping_*`: sempre realiza o wrap-around, tanto em debug quanto em release, como em `wrapping_add`.
> - `checked_*`: retorna `None` se ocorrer estouro.
> - `overflowing_*`: retorna o valor resultante e um `bool` indicando se houve estouro.
> - `saturating_*`: limita o valor ao mínimo ou máximo permitido pelo tipo.
>
> Esses métodos permitem que você escolha explicitamente como o programa deve se comportar quando ocorre um estouro de inteiro, tornando o código mais seguro e previsível.

### Tipos de ponto flutuante

Rust também possui dois tipos primitivos para números de ponto flutuante, que são números com casas decimais. Os tipos de ponto flutuante do Rust são `f32` e `f64`, que possuem 32 bits e 64 bits de tamanho, respectivamente.

O tipo padrão é o `f64`, porque nos CPUs modernos ele tem praticamente a mesma performance que o `f32`, mas oferece maior precisão. Todos os tipos de ponto flutuante em Rust são com sinal, ou seja, podem representar tanto valores positivos quanto negativos.

Veja um exemplo que mostra números de ponto flutuante em uso:

Arquivo: src/main.rs
```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

Os números de ponto flutuante em Rust são representados de acordo com o padrão IEEE-754, que é amplamente utilizado para definir como esse tipo de número é armazenado e processado pelos computadores.

### Operações numéricas

Rust oferece suporte às operações matemáticas básicas que você já espera para todos os tipos numéricos: adição, subtração, multiplicação, divisão e resto da divisão. No caso da divisão entre inteiros, o resultado é truncado em direção a zero, ou seja, o valor decimal é descartado e o resultado é o inteiro mais próximo de zero.

O código a seguir mostra como usar cada uma dessas operações em uma instrução `let`:

Arquivo: src/main.rs
```rust
fn main() {
    // adição
    let sum = 5 + 10;

    // subtração
    let difference = 95.5 - 4.3;

    // multiplicação
    let product = 4 * 30;

    // divisão
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // O resultado é -1

    // resto da divisão
    let remainder = 43 % 5;
}
```

Cada expressão nessas instruções utiliza um operador matemático e é avaliada para um único valor, que em seguida é associado a uma variável. O [Apêndice B](#) contém uma lista completa de todos os operadores que o Rust disponibiliza.

### O tipo booleano

Assim como na maioria das outras linguagens de programação, o tipo booleano em Rust possui apenas dois valores possíveis: `true` e `false`. Os valores booleanos ocupam um byte de memória. Em Rust, o tipo booleano é representado pelo tipo `bool`.

Por exemplo:

Arquivo: src/main.rs
```rust
fn main() {
    let t = true;

    let f: bool = false; // com anotação explícita de tipo
}
```

A principal forma de utilizar valores booleanos é por meio de estruturas condicionais, como uma expressão `if`. Veremos como as expressões `if` funcionam em Rust na seção [fluxo de controle](#).

### O tipo caractere

O tipo `char` em Rust é o tipo alfabético mais primitivo da linguagem. A seguir estão alguns exemplos de como declarar valores do tipo `char`:

Arquivo: src/main.rs
```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // com anotação explícita de tipo
    let heart_eyed_cat = '😻';
}
```

Note que os literais do tipo `char` são definidos usando aspas simples, diferente dos literais de string, que utilizam aspas duplas. O tipo `char` em Rust ocupa 4 bytes de memória e representa um valor escalar Unicode, o que significa que ele pode representar muito mais do que apenas caracteres ASCII.

Letras acentuadas, caracteres chineses, japoneses e coreanos, emojis e até espaços de largura zero são todos valores válidos do tipo `char` em Rust. Os valores escalares Unicode variam de `U+0000` a `U+D7FF` e de `U+E000` a `U+10FFFF`, inclusive.

No entanto, o conceito de "caractere" não é exatamente bem definido no Unicode, então a intuição humana sobre o que é um "caractere" pode não corresponder exatamente ao que um `char` representa em Rust. Vamos discutir esse assunto em mais detalhes na seção [armazenando texto codificado em UTF-8 com Strings](#), no capítulo 8.

## Tipos compostos

Os tipos compostos podem agrupar vários valores em um único tipo. Rust possui dois tipos compostos primitivos: `tuples` e `arrays`.

### O tipo tupla *(tuple)*

Uma tupla é uma forma geral de agrupar vários valores, possivelmente de tipos diferentes, em um único tipo composto. As tuplas possuem um tamanho fixo: depois de declaradas, elas não podem crescer nem diminuir.

Criamos uma tupla escrevendo uma lista de valores separados por vírgula dentro de parênteses. Cada posição da tupla possui um tipo, e os tipos dos diferentes valores não precisam ser iguais. No exemplo a seguir, adicionamos anotações de tipo opcionais:

Arquivo: src/main.rs
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

A variável `tup` fica associada à tupla inteira, pois uma tupla é considerada um único elemento composto. Para obter os valores individuais de uma tupla, podemos usar *pattern matching* para desestruturar o valor da tupla, como neste exemplo:

Arquivo: src/main.rs
```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {y}");
}
```

Esse programa primeiro cria uma tupla e a associa à variável `tup`. Em seguida, ele usa um padrão com `let` para pegar `tup` e transformá-la em três variáveis separadas: `x`, `y` e `z`. Isso é chamado de desestruturação, pois divide a tupla única em três partes. Por fim, o programa imprime o valor de `y`, que é `6.4`.

Também é possível acessar um elemento da tupla diretamente usando um ponto (`.`) seguido do índice do valor que queremos acessar. Por exemplo:

Arquivo: src/main.rs
```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

Esse programa cria a tupla `x` e depois acessa cada elemento usando seus respectivos índices. Assim como na maioria das linguagens de programação, o primeiro índice de uma tupla é `0`.

A tupla sem nenhum valor possui um nome especial: *unit*. Esse valor e seu tipo correspondente são escritos como `()` e representam um valor vazio ou um tipo de retorno vazio. Expressões retornam implicitamente o valor *unit* quando não retornam nenhum outro valor.

### O tipo array

Outra forma de ter uma coleção de vários valores é usando um `array`. Diferente de uma tupla, todos os elementos de um array devem ter o mesmo tipo. Além disso, ao contrário dos arrays em algumas outras linguagens, os arrays em Rust possuem tamanho fixo.

Escrevemos os valores de um array como uma lista de valores separados por vírgula, dentro de colchetes:

Arquivo: src/main.rs
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

Arrays são úteis quando você quer que seus dados sejam alocados na stack, assim como os outros tipos que vimos até agora, em vez da heap (falaremos mais sobre stack e heap no [Capítulo 4](#)), ou quando você quer garantir que sempre haverá um número fixo de elementos.

No entanto, um array não é tão flexível quanto o tipo `vector`. Um vector é um tipo de coleção semelhante, fornecido pela biblioteca padrão, que pode crescer ou diminuir de tamanho, pois seu conteúdo fica armazenado na heap. Se você não tiver certeza se deve usar um array ou um vector, na maioria dos casos a melhor escolha será um vector. O [Capítulo 8](#) aborda os vectors com mais detalhes.

Ainda assim, arrays são mais apropriados quando você sabe que o número de elementos não precisará mudar. Por exemplo, se você estivesse usando os nomes dos meses em um programa, provavelmente usaria um array em vez de um vector, pois sabe que ele sempre conterá exatamente 12 elementos:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Você define o tipo de um array usando colchetes, com o tipo de cada elemento, seguido de um ponto e vírgula e, depois, o número de elementos do array, como neste exemplo:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Aqui, `i32` é o tipo de cada elemento. Após o ponto e vírgula, o número `5` indica que o array contém cinco elementos.

Também é possível inicializar um array para que todos os elementos tenham o mesmo valor, informando o valor inicial, seguido de um ponto e vírgula e do tamanho do array, tudo dentro de colchetes, como mostrado a seguir:

```rust
let a = [3; 5];
```

O array chamado `a` conterá cinco elementos, e todos eles serão inicialmente definidos com o valor `3`. Isso é equivalente a escrever `[3, 3, 3, 3, 3]`, mas de uma forma mais curta e mais conveniente.

#### Acesso a elementos de um array

Um array é um único bloco de memória, com tamanho conhecido e fixo, que pode ser alocado na stack. Você pode acessar os elementos de um array usando indexação, como no exemplo a seguir:

Arquivo: src/main.rs
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

Nesse exemplo, a variável chamada `first` receberá o valor `1`, pois esse é o valor que está no índice `[0]` do array. Já a variável chamada `second` receberá o valor `2`, que está no índice `[1]` do array.

#### Acesso inválido a elementos de um array

Vamos ver o que acontece se você tentar acessar um elemento de um array que está fora dos limites dele. Imagine que você execute o código a seguir, semelhante ao jogo de adivinhação do Capítulo 2, para obter um índice do array digitado pelo usuário:

Arquivo: src/main.rs
```rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!("The value of the element at index {index} is: {element}");
}
```

Esse código compila com sucesso. Se você executá-lo usando `cargo run` e digitar `0`, `1`, `2`, `3` ou `4`, o programa imprimirá o valor correspondente ao índice informado no array.

Porém, se você digitar um número que ultrapasse o final do array, como `10`, verá uma saída semelhante a esta:

```
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with RUST_BACKTRACE=1 environment variable to display a backtrace
```

O programa resultou em um erro em tempo de execução no momento em que tentou usar um valor inválido na operação de indexação. O programa foi encerrado com uma mensagem de erro e não executou a instrução final `println!`.

Quando você tenta acessar um elemento usando indexação, o Rust verifica se o índice informado é menor que o tamanho do array. Se o índice for maior ou igual ao tamanho do array, o Rust entra em panic (interrompe a execução do programa com erro). Essa verificação precisa acontecer em tempo de execução, especialmente neste caso, porque o compilador não tem como saber antecipadamente qual valor o usuário vai informar quando o código for executado.

Esse comportamento é um exemplo dos princípios de segurança de memória do Rust em ação. Em muitas linguagens de baixo nível, esse tipo de verificação não é feito, e ao fornecer um índice incorreto, o programa pode acabar acessando uma região de memória inválida. O Rust protege você contra esse tipo de erro encerrando o programa imediatamente, em vez de permitir o acesso indevido à memória e continuar a execução. O [Capítulo 9](#) aborda com mais detalhes o tratamento de erros em Rust e mostra como você pode escrever um código legível e seguro, que não entre em panic nem permita acessos inválidos à memória.