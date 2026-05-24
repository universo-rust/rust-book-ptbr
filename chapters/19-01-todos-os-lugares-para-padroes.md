---
title: "Todos os lugares onde padrões podem ser usados"
chapter_code: 19-01
slug: todos-os-lugares-onde-padroes-podem-ser-usados
---

# Todos os Lugares Onde Padrões Podem Ser Usados

Padrões aparecem em vários lugares em Rust, e você já os usou bastante sem perceber! Esta seção discute todos os lugares onde padrões são válidos.

## Braços de `match`

Como discutido no Capítulo 6, usamos padrões nos braços de expressões `match`. Formalmente, expressões `match` são definidas como a palavra-chave `match`, um valor a casar, e um ou mais braços de match que consistem em um padrão e uma expressão a executar se o valor casar com o padrão daquele braço, assim:

```
match VALOR {
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
}
```

Por exemplo, aqui está a expressão `match` da Listagem 6-5 que casa com um valor `Option<i32>` na variável `x`:

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Os padrões nesta expressão `match` são o `None` e o `Some(i)` à esquerda de cada seta.

Um requisito para expressões `match` é que elas precisam ser exaustivas no sentido de que todas as possibilidades para o valor na expressão `match` devem ser contempladas. Uma forma de garantir que você cobriu todas as possibilidades é ter um padrão catch-all no último braço: Por exemplo, um nome de variável que casa com qualquer valor nunca pode falhar e, portanto, cobre todos os casos restantes.

O padrão particular `_` casa com qualquer coisa, mas nunca liga a uma variável, então é frequentemente usado no último braço de match. O padrão `_` pode ser útil quando você quer ignorar qualquer valor não especificado, por exemplo. Cobriremos o padrão `_` com mais detalhes em [Ignorando valores em um padrão](#ignorando-valores-em-um-padrao) mais adiante neste capítulo.

## Instruções `let`

Antes deste capítulo, havíamos discutido explicitamente apenas o uso de padrões com `match` e `if let`, mas na verdade já usamos padrões em outros lugares também, incluindo em instruções `let`. Por exemplo, considere esta atribuição simples de variável com `let`:

```rust
let x = 5;
```

Toda vez que você usou uma instrução `let` assim, estava usando padrões, embora talvez não tenha percebido! De forma mais formal, uma instrução `let` se parece com isto:

```
let PADRÃO = EXPRESSÃO;
```

Em instruções como `let x = 5;` com um nome de variável no slot PADRÃO, o nome da variável é apenas uma forma particularmente simples de um padrão. Rust compara a expressão com o padrão e atribui quaisquer nomes que encontrar. Então, no exemplo `let x = 5;`, `x` é um padrão que significa “ligue o que casar aqui à variável `x`.” Como o nome `x` é o padrão inteiro, este padrão efetivamente significa “ligue tudo à variável `x`, seja qual for o valor.”

Para ver o aspecto de pattern matching de `let` com mais clareza, considere a Listagem 19-1, que usa um padrão com `let` para desestruturar uma tupla.

**Arquivo: src/main.rs**

```rust
fn main() {
    let (x, y, z) = (1, 2, 3);
}
```

[Listagem 19-1](#listagem-19-1): Usando um padrão para desestruturar uma tupla e criar três variáveis de uma vez

Aqui, casamos uma tupla com um padrão. Rust compara o valor `(1, 2, 3)` com o padrão `(x, y, z)` e vê que o valor casa com o padrão — ou seja, vê que o número de elementos é o mesmo em ambos — então Rust liga `1` a `x`, `2` a `y` e `3` a `z`. Você pode pensar neste padrão de tupla como aninhar três padrões de variável individuais dentro dele.

Se o número de elementos no padrão não corresponder ao número de elementos na tupla, o tipo geral não casará e receberemos um erro de compilador. Por exemplo, a Listagem 19-2 mostra uma tentativa de desestruturar uma tupla com três elementos em duas variáveis, o que não funcionará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let (x, y) = (1, 2, 3);
}
```

[Listagem 19-2](#listagem-19-2): Construindo incorretamente um padrão cujas variáveis não correspondem ao número de elementos na tupla

Tentar compilar este código resulta neste erro de tipo:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

Para corrigir o erro, poderíamos ignorar um ou mais dos valores na tupla usando `_` ou `..`, como você verá na seção [Ignorando valores em um padrão](#ignorando-valores-em-um-padrao). Se o problema é que temos variáveis demais no padrão, a solução é fazer os tipos casarem removendo variáveis para que o número de variáveis seja igual ao número de elementos na tupla.

## Expressões condicionais `if let`

No Capítulo 6, discutimos como usar expressões `if let` principalmente como uma forma mais curta de escrever o equivalente a um `match` que casa apenas um caso. Opcionalmente, `if let` pode ter um `else` correspondente contendo código a executar se o padrão no `if let` não casar.

A Listagem 19-3 mostra que também é possível misturar expressões `if let`, `else if` e `else if let`. Fazer isso nos dá mais flexibilidade do que uma expressão `match`, na qual podemos expressar apenas um valor a comparar com os padrões. Além disso, Rust não exige que as condições em uma série de braços `if let`, `else if` e `else if let` estejam relacionadas entre si.

O código na Listagem 19-3 determina qual cor usar como fundo com base em uma série de verificações de várias condições. Para este exemplo, criamos variáveis com valores fixos que um programa real poderia receber de entrada do usuário.

**Arquivo: src/main.rs**

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

[Listagem 19-3](#listagem-19-3): Misturando `if let`, `else if`, `else if let` e `else`

Se o usuário especificar uma cor favorita, essa cor é usada como fundo. Se nenhuma cor favorita for especificada e hoje for terça-feira, a cor de fundo é verde. Caso contrário, se o usuário especificar sua idade como uma string e conseguirmos fazer o parse dela como número com sucesso, a cor é roxa ou laranja dependendo do valor do número. Se nenhuma dessas condições se aplicar, a cor de fundo é azul.

Esta estrutura condicional nos permite atender requisitos complexos. Com os valores fixos que temos aqui, este exemplo imprimirá `Using purple as the background color`.

Você pode ver que `if let` também pode introduzir novas variáveis que fazem shadow de variáveis existentes da mesma forma que braços de `match` podem: A linha `if let Ok(age) = age` introduz uma nova variável `age` que contém o valor dentro da variante `Ok`, fazendo shadow da variável `age` existente. Isso significa que precisamos colocar a condição `if age > 30` dentro desse bloco: Não podemos combinar essas duas condições em `if let Ok(age) = age && age > 30`. O novo `age` que queremos comparar com 30 não é válido até que o novo escopo comece com a chave.

A desvantagem de usar expressões `if let` é que o compilador não verifica exaustividade, enquanto com expressões `match` ele verifica. Se omitíssemos o último bloco `else` e, portanto, deixássemos de tratar alguns casos, o compilador não nos alertaria sobre o possível bug de lógica.

## Loops condicionais `while let`

Semelhante na construção a `if let`, o loop condicional `while let` permite que um loop `while` execute enquanto um padrão continuar casando. Na Listagem 19-4, mostramos um loop `while let` que espera por mensagens enviadas entre threads, mas neste caso verificando um `Result` em vez de um `Option`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let (tx, rx) = std::sync::mpsc::channel();
    std::thread::spawn(move || {
        for val in [1, 2, 3] {
            tx.send(val).unwrap();
        }
    });

    while let Ok(value) = rx.recv() {
        println!("{value}");
    }
}
```

[Listagem 19-4](#listagem-19-4): Usando um loop `while let` para imprimir valores enquanto `rx.recv()` retorna `Ok`

Este exemplo imprime `1`, `2` e depois `3`. O método `recv` tira a primeira mensagem do lado receptor do channel e retorna um `Ok(value)`. Quando vimos `recv` pela primeira vez no Capítulo 16, desembrulhamos o erro diretamente, ou interagimos com ele como um iterador usando um loop `for`. Como a Listagem 19-4 mostra, porém, também podemos usar `while let`, porque o método `recv` retorna um `Ok` cada vez que uma mensagem chega, enquanto o sender existir, e então produz um `Err` quando o lado sender desconecta.

## Loops `for`

Em um loop `for`, o valor que vem diretamente após a palavra-chave `for` é um padrão. Por exemplo, em `for x in y`, o `x` é o padrão. A Listagem 19-5 demonstra como usar um padrão em um loop `for` para desestruturar, ou separar, uma tupla como parte do loop `for`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{value} is at index {index}");
    }
}
```

[Listagem 19-5](#listagem-19-5): Usando um padrão em um loop `for` para desestruturar uma tupla

O código na Listagem 19-5 imprimirá o seguinte:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/patterns`
a is at index 0
b is at index 1
c is at index 2
```

Adaptamos um iterador usando o método `enumerate` para que ele produza um valor e o índice daquele valor, colocados em uma tupla. O primeiro valor produzido é a tupla `(0, 'a')`. Quando este valor é casado com o padrão `(index, value)`, index será `0` e value será `'a'`, imprimindo a primeira linha da saída.

## Parâmetros de função

Parâmetros de função também podem ser padrões. O código na Listagem 19-6, que declara uma função chamada `foo` que recebe um parâmetro chamado `x` do tipo `i32`, deve ser familiar a esta altura.

**Arquivo: src/main.rs**

```rust
fn foo(x: i32) {
    // code goes here
}
```

[Listagem 19-6](#listagem-19-6): Uma assinatura de função usando padrões nos parâmetros

A parte `x` é um padrão! Como fizemos com `let`, poderíamos casar uma tupla nos argumentos de uma função com o padrão. A Listagem 19-7 separa os valores em uma tupla conforme a passamos para uma função.

**Arquivo: src/main.rs**

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({x}, {y})");
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

[Listagem 19-7](#listagem-19-7): Uma função com parâmetros que desestruturam uma tupla

Este código imprime `Current location: (3, 5)`. Os valores `&(3, 5)` casam com o padrão `&(x, y)`, então `x` é o valor `3` e `y` é o valor `5`.

Também podemos usar padrões em listas de parâmetros de closures da mesma forma que em listas de parâmetros de funções, porque closures são semelhantes a funções, como discutido no Capítulo 13.

Até este ponto, você viu várias formas de usar padrões, mas padrões não funcionam da mesma maneira em todo lugar onde podemos usá-los. Em alguns lugares, os padrões devem ser irrefutáveis; em outras circunstâncias, podem ser refutáveis. Discutiremos esses dois conceitos a seguir.
