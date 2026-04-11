---
title: "Funções"
chapter_code: 03-03
slug: funcoes
---

# Funções

Funções são muito comuns em código Rust. Você já viu uma das funções mais importantes da linguagem: a função `main`, que é o ponto de entrada de muitos programas. Você também já viu a palavra-chave `fn`, que permite declarar novas funções.

O código Rust utiliza _snake case_ como estilo convencional para nomes de funções e variáveis, em que todas as letras são minúsculas e as palavras são separadas por underscores (`_`). Aqui está um programa que contém um exemplo de definição de função:

**Arquivo: src/main.rs**

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

_Listagem 1-1: exemplo de definição de função em Rust_

Definimos uma função em Rust usando `fn`, seguido do nome da função e de um conjunto de parênteses. As chaves `{}` indicam ao compilador onde o corpo da função começa e termina.

Podemos chamar qualquer função que tenhamos definido usando seu nome seguido de parênteses. Como `another_function` está definida no programa, ela pode ser chamada de dentro da função `main`. Note que definimos `another_function` depois da função `main` no código-fonte; também poderíamos tê-la definido antes. O Rust não se importa com a ordem em que você define suas funções, apenas que elas estejam definidas em algum lugar dentro de um _escopo_ que possa ser visto por quem as chama.

Vamos criar um novo projeto binário chamado `functions` para explorar funções com mais detalhes. Coloque o exemplo de `another_function` em `src/main.rs` e execute-o. Você deverá ver a seguinte saída:

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/functions`
Hello, world!
Another function.
```

As linhas são executadas na ordem em que aparecem dentro da função `main`. Primeiro, a mensagem `"Hello, world!"` é impressa, e então `another_function` é chamada e sua mensagem é impressa.

## Parâmetros

Podemos definir funções para receber _parâmetros_, que são variáveis especiais que fazem parte da assinatura da função. Quando uma função tem parâmetros, você pode fornecer valores concretos para esses parâmetros. Tecnicamente, os valores concretos são chamados de _argumentos_, mas, na conversa cotidiana, as pessoas tendem a usar os termos parâmetro e argumento de forma intercambiável, tanto para se referir às variáveis na definição da função quanto aos valores passados quando chamamos a função.

Nesta versão de `another_function`, adicionamos um parâmetro:

**Arquivo: src/main.rs**

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}
```

Tente executar este programa; você deverá obter a seguinte saída:

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.21s
     Running `target/debug/functions`
The value of x is: 5
```

A declaração de `another_function` possui um parâmetro chamado `x`. O tipo de `x` é especificado como `i32`. Quando passamos o valor `5` para `another_function`, a macro `println!` coloca `5` no lugar do par de chaves que contém `x` na string de formatação.

Em assinaturas de funções, você **precisa declarar o tipo de cada parâmetro**. Essa é uma decisão intencional no design do Rust: exigir anotações de tipo nas definições de função significa que o compilador quase nunca precisará que você as use em outros lugares do código para descobrir que tipo você quis dizer. Além disso, o compilador consegue fornecer mensagens de erro mais úteis se souber quais tipos a função espera.

Ao definir múltiplos parâmetros, separe as declarações de parâmetros com vírgulas, assim:

**Arquivo: src/main.rs**

```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```

Este exemplo cria uma função chamada `print_labeled_measurement` com dois parâmetros. O primeiro parâmetro se chama `value` e é do tipo `i32`. O segundo parâmetro se chama `unit_label` e é do tipo `char`. A função então imprime um texto contendo tanto o `value` quanto o `unit_label`.

Vamos tentar executar este código. Substitua o programa que está atualmente no arquivo _src/main.rs_ do seu projeto de funções pelo exemplo anterior e execute-o usando o comando `cargo run`.

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/functions`
The measurement is: 5h
```

Como chamamos a função passando `5` como valor de `value` e `'h'` como valor de `unit_label`, a saída do programa contém esses valores.

## Declarações e expressões

Os corpos das funções em Rust são compostos por uma série de declarações que podem terminar, opcionalmente, em uma expressão. Até agora, as funções que vimos não incluíam uma expressão final, mas você já viu expressões sendo usadas dentro de declarações. Como o Rust é uma linguagem baseada em expressões, essa distinção é muito importante de entender. Outras linguagens não fazem essa separação da mesma forma, então vamos analisar o que são declarações e expressões, e como as diferenças entre elas afetam o corpo das funções.

- _Declarações_ são comandos que executam alguma ação e não retornam um valor.
- As _expressões_ avaliam e resultam em um valor.

Vamos ver alguns exemplos.

Na verdade, nós já usamos declarações e expressões antes. Criar uma variável e atribuir um valor a ela usando a palavra-chave `let` é uma declaração. No exemplo da Listagem 3-1, `let y = 6;` é uma declaração.

**Arquivo: src/main.rs**

```rust
fn main() {
    let y = 6;
}
```

_Listagem 3-1: Exemplo de declaração da função `main` contendo uma única instrução._

Definições de funções também são declarações; ou seja, o exemplo acima inteiro é, ele próprio, uma declaração. (Como veremos em breve, chamar uma função não é uma declaração.)

Declarações **não retornam valores**. Portanto, você não pode atribuir uma declaração `let` a outra variável, como o código a seguir tenta fazer; isso vai gerar um erro:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = (let y = 6);
}
```

Ao executar este programa, o erro que você receberá será semelhante a este:

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found `let` statement
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^
  |
  = note: only supported directly in conditions of `if` and `while` expressions

warning: unnecessary parentheses around assigned value
 --> src/main.rs:2:13
  |
2 |     let x = (let y = 6);
  |             ^         ^
  |
  = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
  |
2 -     let x = (let y = 6);
2 +     let x = let y = 6;
  |

warning: `functions` (bin "functions") generated 1 warning
error: could not compile `functions` (bin "functions") due to 1 previous error; 1 warning emitted
```

A instrução `let y = 6` não retorna um valor, então não há nada para o qual `x` possa se vincular. Isso é diferente do que acontece em outras linguagens, como C e Ruby, nas quais a atribuição retorna o valor atribuído. Nessas linguagens, você pode escrever `x = y = 6` e fazer com que tanto `x` quanto `y` tenham o valor `6`; esse não é o caso em Rust.

Expressões resultam em um valor e constituem a maior parte do restante do código que você escreverá em Rust. Considere uma operação matemática, como `5 + 6`, que é uma expressão que resulta no valor `11`. Expressões podem fazer parte de instruções: no exemplo 3-1, o `6` na instrução `let y = 6;` é uma expressão que resulta no valor `6`. Chamar uma função é uma expressão. Chamar uma macro é uma expressão. Um novo bloco de escopo criado com chaves é uma expressão, por exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

Essa expressão:

```rust
{
    let x = 3;
    x + 1
}
```

é um bloco que, neste caso, resulta em `4`. Esse valor é associado a `y` como parte da instrução `let`. Observe a linha `x + 1` sem ponto e vírgula no final, que é diferente da maioria das linhas que você viu até agora. Expressões não incluem ponto e vírgula no final. Se você adicionar um ponto e vírgula ao final de uma expressão, você a transforma em uma instrução e, portanto, ela não retornará um valor. Lembre-se disso ao explorar valores de retorno de funções e expressões a seguir.

## Funções com valores de retorno

As funções podem retornar valores para o código que as chama. Não damos nomes aos valores de retorno, mas precisamos declarar o tipo deles depois de uma seta (`->`). Em Rust, o valor retornado por uma função é, na prática, o mesmo que o valor da última expressão dentro do bloco que forma o corpo da função. É possível encerrar a execução de uma função mais cedo usando a palavra-chave `return` e especificando um valor. No entanto, na maioria dos casos, as funções retornam implicitamente o resultado da última expressão. Aqui está um exemplo de uma função que retorna um valor.

**Arquivo: src/main.rs**

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

Não há chamadas de função, macros ou mesmo instruções `let` dentro da função `five` — apenas o número `5` por si só. Essa é uma função perfeitamente válida em Rust. Observe que o tipo de retorno da função também é especificado como `-> i32`. Tente executar este código; a saída deve ser semelhante a esta:

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/functions`
The value of x is: 5
```

O `5` em `five` é o valor de retorno da função, e é por isso que o tipo de retorno é `i32`. Vamos examinar isso com mais detalhes. Há dois pontos importantes: Primeiro, a linha `let x = five();` mostra que estamos usando o valor de retorno de uma função para inicializar uma variável. Como a função `five` retorna `5`, essa linha é equivalente à seguinte:

```rust
let x = 5;
```

Em segundo lugar, a função `five` não tem parâmetros e define o tipo do valor de retorno, mas o corpo da função é um `5` isolado, sem ponto e vírgula, porque é uma expressão cujo valor queremos retornar.

Vejamos outro exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

A compilação deste código produzirá um erro, conforme mostrado abaixo:

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error[E0308]: mismatched types
 --> src/main.rs:7:24
  |
7 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
8 |     x + 1;
  |          - help: remove this semicolon to return this value

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions` (bin "functions") due to 1 previous error
```

A mensagem de erro `mismatched types` revela o problema central do código. A definição da função `plus_one` diz que ela deve retornar um `i32`, mas as instruções não produzem um valor — elas resultam em `()`, o tipo unitário. Isso significa que nada está sendo retornado, o que contradiz a definição da função e gera o erro. O compilador do Rust sugere uma correção: remover o ponto e vírgula. Fazendo isso, a expressão passa a produzir um valor, e o erro desaparece.