---
title: "Fluxo de controle"
chapter_code: 03-05
slug: fluxo-de-controle
---

# Fluxo de controle

A capacidade de executar algum código dependendo se uma condição é verdadeira e a capacidade de executar código repetidamente enquanto uma condição for verdadeira são elementos básicos na maioria das linguagens de programação. As estruturas mais comuns que permitem controlar o fluxo de execução do código Rust são as expressões `if` e os loops.

## Expressões if

Uma expressão `if` permite que você direcione seu código dependendo de condições. Você fornece uma condição e então declara: "Se esta condição for atendida, execute este bloco de código. Se a condição não for atendida, não execute este bloco de código."

Crie um novo projeto chamado _branches_ no diretório _projects_ para explorar a expressão `if`. No arquivo `src/main.rs`, insira o seguinte:

**Arquivo: src/main.rs**

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

Todas as expressões `if` começam com a palavra-chave `if`, seguida por uma condição. Neste caso, a condição verifica se a variável `number` tem um valor menor que 5. O bloco de código a ser executado se a condição for verdadeira é colocado logo após a condição, dentro de chaves.

Os blocos de código associados às condições em expressões `if` às vezes são chamados de braços (_arms_), assim como os braços nas expressões `match`.

Opcionalmente, também podemos incluir uma expressão `else` para fornecer um bloco alternativo de código caso a condição seja falsa. Se você não fornecer um `else` e a condição for falsa, o programa simplesmente ignora o bloco do `if` e continua a execução.

Ao executar o código acima, você verá a seguinte saída:

```bash
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was true
```

Se alterarmos o valor de `number` para:

```rust
let number = 7;
```

A saída será:

```bash
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was false
```

A condição de um `if` deve ser obrigatoriamente do tipo `bool`. Caso contrário, ocorrerá um erro de compilação.

**Arquivo: src/main.rs**

```rust
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```

Desta vez, a condição do `if` passa a valer `3`, e o Rust lança um erro.

```bash
$ cargo run
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected `bool`, found integer
```

O erro indica que o Rust esperava um booleano, mas recebeu um inteiro. Ao contrário de linguagens como Ruby e JavaScript, o Rust não tenta converter automaticamente tipos não booleanos em booleanos. Você deve ser explícito e sempre fornecer um booleano como condição do `if`. Se quisermos que o bloco de código `if` seja executado somente quando um número for diferente de 0, por exemplo, podemos alterar a expressão `if` para o seguinte:

**Arquivo: src/main.rs**

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```

Executar este código imprimirá `number was something other than zero`.

## Lidando com múltiplas condições com else if

Você pode usar múltiplas condições combinando `if` e `else` em uma expressão `else if`. Por exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

Este programa possui quatro caminhos possíveis. Após executá-lo, você deverá ver a seguinte saída.

```bash
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
number is divisible by 3
```

Quando este programa é executado, ele verifica cada expressão `if` por sua vez e executa o primeiro bloco `else` para o qual a condição é avaliada como verdadeira. Observe que, embora `6` seja divisível por `2`, não vemos a mensagem `number is divisible by 2`, nem vemos o texto `number is not divisible by 4, 3, or 2` do bloco `else`. Isso ocorre porque o Rust executa o bloco apenas para a primeira condição verdadeira e, uma vez encontrada, não verifica as demais.

Usar muitas expressões `else if` pode sobrecarregar seu código; portanto, se você tiver mais de uma, talvez seja interessante refatorar seu código. O Capítulo 6 descreve uma poderosa construção de ramificação em Rust chamada `match` para esses casos.

## Usando if em uma instrução let

Como `if` é uma expressão, podemos usá-la no lado direito de uma instrução `let` para atribuir o resultado a uma variável, como no exemplo 3-2.

**Arquivo: src/main.rs**

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

_Listagem 3-2: Atribuindo o resultado de uma expressão `if` a uma variável_

A variável `number` será associada a um valor com base no resultado da expressão condicional (`if`). Execute este código para ver o que acontece:

```bash
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/branches`
The value of number is: 5
```

Lembre-se de que blocos de código resultam no valor da última expressão contida neles, e números sozinhos também são expressões. Neste caso, o valor de toda a expressão `if` depende de qual bloco de código será executado. Isso significa que os valores produzidos por cada ramificação (ou "braço") do `if` devem ser do mesmo tipo. No Exemplo 3-2, os resultados tanto do `if` quanto do `else` eram inteiros do tipo `i32`. Se os tipos forem diferentes, como no exemplo a seguir, ocorrerá um erro:

**Arquivo: src/main.rs**

```rust
fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {number}");
}
```

Ao tentar compilar este código, receberemos um erro. Os blocos `if` e `else` possuem tipos diferentes, e o Rust indica exatamente onde encontrar o problema no programa:

```bash
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:44
  |
4 |     let number = if condition { 5 } else { "six" };
  |                                 -          ^^^^^ expected integer, found `&str`
  |                                 |
  |                                 expected because of this

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches` (bin "branches") due to 1 previous error
```

A expressão no bloco `if` resulta em um inteiro, e a expressão no bloco `else` resulta em uma string. Isso não funcionará, porque as variáveis devem ter um único tipo, e o Rust precisa saber definitivamente em tempo de compilação qual é o tipo da variável `number`. Conhecer o tipo de `number` permite que o compilador verifique se o tipo é válido em todos os lugares onde usamos `number`. Rust não seria capaz de fazer isso se o tipo de `number` fosse determinado apenas em tempo de execução; o compilador seria mais complexo e faria menos garantias sobre o código se tivesse que controlar múltiplos tipos hipotéticos para qualquer variável.

## Laços de repetição

Muitas vezes, é útil executar um bloco de código mais de uma vez. Para essa tarefa, o Rust oferece diversos tipos de loops (laços de repetição), que executam o código dentro do corpo do loop até o fim e, imediatamente, reiniciam a partir do começo. Para experimentarmos os laços de repetição, vamos criar um novo projeto chamado `loops`.

O Rust possui três tipos de laços: `loop`, `while` e `for`. Vamos testar cada um deles.

## Repetindo código com loop

A palavra-chave `loop` diz ao Rust para executar um bloco de código repetidamente, indefinidamente ou até que você o interrompa explicitamente.

Como exemplo, altere o arquivo `src/main.rs` no diretório `loops` para ficar assim:

**Arquivo: src/main.rs**

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

Quando executarmos esse programa, veremos a palavra `again!` sendo impressa continuamente, até que interrompamos o programa manualmente. A maioria dos terminais suporta o atalho de teclado `Ctrl`+`C` para interromper um programa que está preso em um loop infinito. Experimente:

```bash
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

O símbolo `^C` representa o momento em que você pressionou `Ctrl`+`C`.

Você pode ou não ver a palavra `again!` sendo impressa após o `^C`, dependendo de onde o código estava no loop no momento em que recebeu o sinal de interrupção.

Felizmente, o Rust também oferece uma forma de sair de um loop usando código. Você pode colocar a palavra-chave `break` dentro do loop para dizer ao programa quando ele deve parar de executar o laço. Lembre-se de que fizemos isso no jogo de adivinhação, na seção "Saindo após um palpite correto" do Capítulo 2, para sair do programa quando o usuário acertava o número correto.

Também usamos `continue` no jogo de adivinhação, que dentro de um loop diz ao programa para pular o restante do código da iteração atual e ir diretamente para a próxima iteração.

## Retornando valores de loops

Um dos usos de um loop é tentar novamente uma operação que pode falhar, como verificar se uma thread terminou seu trabalho. Você também pode precisar passar o resultado dessa operação para fora do loop, para o restante do seu código. Para fazer isso, você pode adicionar o valor que deseja retornar após a expressão `break` usada para encerrar o loop; esse valor será retornado pelo loop e poderá ser utilizado, como mostrado a seguir:

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

Antes do loop, declaramos uma variável chamada `counter` e a inicializamos com `0`. Em seguida, declaramos uma variável chamada `result` para armazenar o valor retornado pelo loop. A cada iteração do loop, somamos `1` à variável `counter` e verificamos se ela é igual a `10`. Quando isso acontece, usamos a palavra-chave `break` com o valor `counter * 2`. Após o loop, usamos um ponto e vírgula para finalizar a instrução que atribui o valor a `result`. Por fim, imprimimos o valor de `result`, que nesse caso é `20`.

Você também pode usar `return` de dentro de um loop. Enquanto `break` sai apenas do loop atual, `return` sempre sai da função atual.

## Resolvendo ambiguidades com rótulos de loop

Se você tiver loops dentro de outros loops, `break` e `continue` se aplicam ao loop mais interno naquele ponto. Opcionalmente, você pode especificar um rótulo de loop em um loop e então usá-lo com `break` ou `continue` para indicar que essas palavras-chave devem se aplicar ao loop rotulado, em vez do loop mais interno.

Os rótulos de loop devem começar com uma aspa simples. Veja um exemplo com dois loops aninhados:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

O `loop` externo tem o rótulo `'counting_up` e fará a contagem crescente de 0 a 2. O `loop` interno, sem rótulo, fará a contagem decrescente de 10 a 9. O primeiro `break` sem rótulo encerrará apenas o `loop` interno. O comando `break 'counting_up;` encerrará o laço externo. Este código imprime:

```bash
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.58s
     Running `target/debug/loops`
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

## Simplificando loops condicionais com while

Um programa frequentemente precisa avaliar uma condição dentro de um laço. Enquanto a condição for verdadeira, o laço continua sendo executado. Quando a condição deixa de ser verdadeira, o programa chama o comando `break`, interrompendo o laço. É possível implementar esse comportamento usando uma combinação de laços `loop`, `if`, `else` e `break`; você pode experimentar isso agora mesmo em um programa, se quiser. No entanto, esse padrão é tão comum que Rust possui uma construção de linguagem integrada para ele, chamada de laço `while`. No exemplo 3-3, usamos `while` para executar o programa três vezes, contando decrescentemente a cada iteração e, em seguida, após o laço, para imprimir uma mensagem e encerrar o programa.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

_Listagem 3-3: Usando um `while` para executar código enquanto uma condição é verdadeira_

Essa construção elimina muitos aninhamentos que seriam necessários se você usasse `loops`, `if`, `else` e `break`, e é mais clara. Enquanto uma condição for avaliada como verdadeira, o código é executado; caso contrário, ele sai do `loop`.

## Percorrendo uma coleção com for

Você pode optar por usar a estrutura `while` para percorrer os elementos de uma coleção, como um array. Por exemplo, o loop na Listagem 3-4 imprime cada elemento do array `a`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}
```

_Listagem 3-4: Percorrendo cada elemento de uma coleção usando um loop `while`_

Neste código, a contagem é feita percorrendo os elementos do array. Começa no índice 0 e repete o processo até atingir o índice final do array (ou seja, quando a condição `index < 5` não for mais verdadeira). Executar este código imprimirá todos os elementos do array.

```bash
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

Todos os cinco valores do array aparecem no terminal, como esperado. Mesmo que o `index` atinja o valor `5` em algum momento, o loop para de executar antes de tentar obter um sexto valor do array.

No entanto, essa abordagem é propensa a erros; podemos fazer com que o programa entre em pânico se o valor do índice ou a condição de teste estiverem incorretos. Por exemplo, se você alterar a definição de um array para ter quatro elementos, mas esquecer de atualizar a condição para `while index < 4`, o código entrará em pânico. Além disso, é lento, porque o compilador adiciona código em tempo de execução para realizar a verificação condicional se o índice está dentro dos limites do array em cada iteração do loop.

Como alternativa mais concisa, você pode usar um laço `for` e executar algum código para cada item em uma coleção. Um laço `for` se parece com o código na Listagem 3-5.

**Arquivo: src/main.rs**

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```

_Listagem 3-5: Percorrendo cada elemento do array usando um loop `for`_

Ao executarmos este código, veremos a mesma saída que na Listagem 3-4. O ponto principal aqui é o ganho em segurança, eliminando bugs causados por acessos fora dos limites (out-of-bounds) ou iterações incompletas. O código de máquina gerado pelo `for` também pode ser mais eficiente, pois o índice não precisa ser comparado ao comprimento do array a cada iteração.

Usando o laço `for`, você não precisaria se lembrar de alterar nenhum outro código se mudasse o número de valores na matriz, como aconteceria com o método usado na Listagem 3-4.

A segurança e a concisão do `for` o torna a estrutura de loop mais usada em Rust. Mesmo em situações em que você deseja executar um código um certo número de vezes, como no exemplo de contagem regressiva que usou um loop `while` na Listagem 3-3, a maioria dos programadores Rust usaria um loop `for`. A maneira de fazer isso seria usar um `Range`, fornecido pela biblioteca padrão, que gera todos os números em sequência, começando de um número e terminando antes de outro.

Eis como ficaria a contagem regressiva usando um loop `for` e outro método que ainda não abordamos, `rev`, para inverter o intervalo.

**Arquivo: src/main.rs**

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```

Este código é um pouco mais elegante, não é?

## Resumo

Você conseguiu! Este foi um capítulo extenso. Você aprendeu sobre variáveis, tipos de dados escalares e compostos, funções, comentários, expressões `if` e loops.

Para praticar os conceitos discutidos neste capítulo, tente construir programas que façam o seguinte:

- Converter temperaturas entre Fahrenheit e Celsius.
- Gerar o n-ésimo número da sequência de Fibonacci.
- Imprimir a letra da canção natalina "The Twelve Days of Christmas", aproveitando a repetição presente na música.

Quando estiver pronto para avançar, falaremos sobre um conceito do Rust que não existe comumente em outras linguagens de programação: ownership.