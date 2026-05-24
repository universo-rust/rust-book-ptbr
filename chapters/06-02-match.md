---
title: "A construção de fluxo de controle `match`"
chapter_code: 06-02
slug: match
---

# A Construção de Fluxo de Controle `match`

O Rust tem uma construção de fluxo de controle extremamente poderosa chamada `match` que permite comparar um valor contra uma série de padrões e então executar código com base em qual padrão corresponde. Padrões podem ser compostos de valores literais, nomes de variáveis, curingas e muitas outras coisas; o Capítulo 19 cobre todos os diferentes tipos de padrões e o que fazem. O poder de `match` vem da expressividade dos padrões e do fato de o compilador confirmar que todos os casos possíveis são tratados.

Pense em uma expressão `match` como uma máquina de classificar moedas: as moedas deslizam por um trilho com furos de tamanhos variados ao longo dele, e cada moeda cai no primeiro furo em que couber. Da mesma forma, valores passam por cada padrão em um `match`, e no primeiro padrão em que o valor “cabe”, o valor cai no bloco de código associado para ser usado durante a execução.

Falando em moedas, vamos usá-las como exemplo com `match`! Podemos escrever uma função que recebe uma moeda americana desconhecida e, de forma semelhante à máquina de contagem, determina qual moeda é e retorna seu valor em centavos, como mostrado na Listagem 6-3.

**Arquivo: src/main.rs**

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {}
```

<a id="listagem-6-3"></a>

[Listagem 6-3](#listagem-6-3): Um enum e uma expressão `match` que tem as variantes do enum como seus padrões

Vamos analisar o `match` na função `value_in_cents`. Primeiro, listamos a palavra-chave `match` seguida de uma expressão, que neste caso é o valor `coin`. Isso parece muito semelhante a uma expressão condicional usada com `if`, mas há uma grande diferença: com `if`, a condição precisa avaliar para um valor booleano, mas aqui pode ser de qualquer tipo. O tipo de `coin` neste exemplo é o enum `Coin` que definimos na primeira linha.

Em seguida vêm os braços do `match`. Um braço tem duas partes: um padrão e algum código. O primeiro braço aqui tem um padrão que é o valor `Coin::Penny` e então o operador `=>` que separa o padrão e o código a executar. O código neste caso é apenas o valor `1`. Cada braço é separado do próximo por uma vírgula.

Quando a expressão `match` é executada, ela compara o valor resultante com o padrão de cada braço, em ordem. Se um padrão corresponder ao valor, o código associado a esse padrão é executado. Se esse padrão não corresponder ao valor, a execução continua para o próximo braço, como em uma máquina de classificar moedas. Podemos ter quantos braços precisarmos: na Listagem 6-3, nosso `match` tem quatro braços.

O código associado a cada braço é uma expressão, e o valor resultante da expressão no braço correspondente é o valor retornado para toda a expressão `match`.

Normalmente não usamos chaves se o código do braço do `match` for curto, como na Listagem 6-3, onde cada braço apenas retorna um valor. Se quiser executar várias linhas de código em um braço do `match`, deve usar chaves, e a vírgula após o braço torna-se opcional. Por exemplo, o código a seguir imprime “Lucky penny!” sempre que o método for chamado com um `Coin::Penny`, mas ainda retorna o último valor do bloco, `1`:

**Arquivo: src/main.rs**

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {}
```

### Padrões que fazem bind a valores

Outro recurso útil dos braços do `match` é que eles podem fazer bind às partes dos valores que correspondem ao padrão. É assim que podemos extrair valores de variantes de enum.

Como exemplo, vamos alterar uma de nossas variantes de enum para conter dados dentro dela. De 1999 a 2008, os Estados Unidos cunharam moedas de 25 centavos com designs diferentes para cada um dos 50 estados em um dos lados. Nenhuma outra moeda recebeu designs de estados, então apenas as moedas de 25 centavos têm esse valor extra. Podemos adicionar essa informação ao nosso `enum` alterando a variante `Quarter` para incluir um valor `UsState` armazenado dentro dela, o que fizemos na Listagem 6-4.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)] // para podermos inspecionar o estado em um instante
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

<a id="listagem-6-4"></a>

[Listagem 6-4](#listagem-6-4): Um enum `Coin` em que a variante `Quarter` também contém um valor `UsState`

Imagine que um amigo está tentando coletar as moedas de 25 centavos dos 50 estados. Enquanto separamos o troco por tipo de moeda, também anunciaremos o nome do estado associado a cada moeda de 25 centavos, para que, se for uma que nosso amigo não tem, ele possa adicioná-la à coleção.

Na expressão `match` deste código, adicionamos uma variável chamada `state` ao padrão que corresponde a valores da variante `Coin::Quarter`. Quando um `Coin::Quarter` corresponder, a variável `state` fará bind ao valor do estado daquela moeda. Então, podemos usar `state` no código daquele braço, assim:

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}
```

Se chamássemos `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` seria `Coin::Quarter(UsState::Alaska)`. Quando comparamos esse valor com cada um dos braços do `match`, nenhum corresponde até chegarmos a `Coin::Quarter(state)`. Nesse ponto, o binding de `state` será o valor `UsState::Alaska`. Podemos então usar esse binding na expressão `println!`, obtendo assim o valor interno do estado da variante `Quarter` do enum `Coin`.

### O padrão `match` de `Option<T>`

Na seção anterior, queríamos obter o valor interno `T` do caso `Some` ao usar `Option<T>`; também podemos lidar com `Option<T>` usando `match`, como fizemos com o enum `Coin`! Em vez de comparar moedas, compararemos as variantes de `Option<T>`, mas a forma como a expressão `match` funciona permanece a mesma.

Digamos que queremos escrever uma função que recebe um `Option<i32>` e, se houver um valor dentro, soma 1 a esse valor. Se não houver valor dentro, a função deve retornar o valor `None` e não tentar realizar nenhuma operação.

Essa função é muito fácil de escrever, graças ao `match`, e ficará como na Listagem 6-5.

**Arquivo: src/main.rs**

```rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

<a id="listagem-6-5"></a>

[Listagem 6-5](#listagem-6-5): Uma função que usa uma expressão `match` em um `Option<i32>`

Vamos examinar a primeira execução de `plus_one` com mais detalhes. Quando chamamos `plus_one(five)`, a variável `x` no corpo de `plus_one` terá o valor `Some(5)`. Então comparamos isso com cada braço do `match`:

```rust
None => None,
```

O valor `Some(5)` não corresponde ao padrão `None`, então continuamos para o próximo braço:

```rust
Some(i) => Some(i + 1),
```

`Some(5)` corresponde a `Some(i)`? Sim! Temos a mesma variante. O `i` faz bind ao valor contido em `Some`, então `i` recebe o valor `5`. O código no braço do `match` é então executado, então somamos 1 ao valor de `i` e criamos um novo valor `Some` com nosso total `6` dentro.

Agora vamos considerar a segunda chamada de `plus_one` na Listagem 6-5, onde `x` é `None`. Entramos no `match` e comparamos com o primeiro braço:

```rust
None => None,
```

Corresponde! Não há valor para somar, então o programa para e retorna o valor `None` no lado direito de `=>`. Como o primeiro braço correspondeu, nenhum outro braço é comparado.

Combinar `match` e enums é útil em muitas situações. Você verá esse padrão muito em código Rust: `match` contra um enum, bind de uma variável aos dados internos e então execução de código com base nisso. É um pouco complicado no início, mas quando você se acostuma, deseja ter isso em todas as linguagens. É consistentemente um dos favoritos dos usuários.

### Matches são exaustivos

Há outro aspecto do `match` que precisamos discutir: os padrões dos braços devem cobrir todas as possibilidades. Considere esta versão da nossa função `plus_one`, que tem um bug e não compilará:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

Não tratamos o caso `None`, então este código causará um bug. Felizmente, é um bug que o Rust sabe detectar. Se tentarmos compilar este código, receberemos este erro:

```bash
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0004]: non-exhaustive patterns: `None` not covered
 --> src/main.rs:3:15
  |
3 |         match x {
  |               ^ pattern `None` not covered
  |
note: `Option<i32>` defined here
 --> /rustc/1159e78c4747b02ef996e55082b704c09b970588/library/core/src/option.rs:593:1
 ::: /rustc/1159e78c4747b02ef996e55082b704c09b970588/library/core/src/option.rs:597:5
  |
  = note: not covered
  = note: the matched value is of type `Option<i32>`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
4 ~             Some(i) => Some(i + 1),
5 ~             None => todo!(),
  |

For more information about this error, try `rustc --explain E0004`.
error: could not compile `enums` (bin "enums") due to 1 previous error
```

O Rust sabe que não cobrimos todos os casos possíveis e até sabe qual padrão esquecemos! Matches em Rust são _exaustivos_: devemos esgotar todas as possibilidades para que o código seja válido. Especialmente no caso de `Option<T>`, quando o Rust nos impede de esquecer de tratar explicitamente o caso `None`, nos protege de assumir que temos um valor quando podemos ter null, tornando impossível o erro de bilhão de dólares discutido antes.

### Padrões catch-all e o placeholder `_`

Usando enums, também podemos tomar ações especiais para alguns valores particulares, mas para todos os outros valores tomar uma ação padrão. Imagine que estamos implementando um jogo em que, se você tirar 3 em uma rolagem de dados, seu jogador não se move, mas ganha um chapéu chique. Se tirar 7, seu jogador perde um chapéu chique. Para todos os outros valores, seu jogador se move esse número de casas no tabuleiro. Aqui está um `match` que implementa essa lógica, com o resultado da rolagem de dados fixo em vez de um valor aleatório, e toda a outra lógica representada por funções sem corpo, porque implementá-las de fato está fora do escopo deste exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
}
```

Para os dois primeiros braços, os padrões são os valores literais `3` e `7`. Para o último braço que cobre todos os outros valores possíveis, o padrão é a variável que escolhemos chamar de `other`. O código que roda para o braço `other` usa a variável passando-a para a função `move_player`.

Este código compila, mesmo que não tenhamos listado todos os valores possíveis que um `u8` pode ter, porque o último padrão corresponderá a todos os valores não listados explicitamente. Esse padrão catch-all atende ao requisito de que `match` deve ser exaustivo. Observe que precisamos colocar o braço catch-all por último porque os padrões são avaliados em ordem. Se tivéssemos colocado o braço catch-all antes, os outros braços nunca seriam executados, então o Rust nos avisará se adicionarmos braços depois de um catch-all!

O Rust também tem um padrão que podemos usar quando queremos um catch-all, mas não queremos _usar_ o valor no padrão catch-all: `_` é um padrão especial que corresponde a qualquer valor e não faz bind a esse valor. Isso diz ao Rust que não vamos usar o valor, então o Rust não nos avisará sobre uma variável não utilizada.

Vamos mudar as regras do jogo: agora, se você tirar qualquer coisa que não seja 3 ou 7, deve rolar novamente. Não precisamos mais usar o valor catch-all, então podemos alterar nosso código para usar `_` em vez da variável chamada `other`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
}
```

Este exemplo também atende ao requisito de exaustividade porque estamos ignorando explicitamente todos os outros valores no último braço; não esquecemos nada.

Por fim, mudaremos as regras do jogo mais uma vez para que nada mais aconteça no seu turno se você tirar qualquer coisa que não seja 3 ou 7. Podemos expressar isso usando o valor unitário (o tipo tupla vazio que mencionamos na seção O tipo tupla do Capítulo 3) como o código que acompanha o braço `_`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
}
```

Aqui, estamos dizendo ao Rust explicitamente que não vamos usar nenhum outro valor que não corresponda a um padrão em um braço anterior, e não queremos executar nenhum código neste caso.

Há mais sobre padrões e matching que cobriremos no Capítulo 19. Por enquanto, passaremos para a sintaxe `if let`, que pode ser útil em situações em que a expressão `match` é um pouco verbosa.
