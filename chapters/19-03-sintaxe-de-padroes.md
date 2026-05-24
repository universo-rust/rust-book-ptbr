---
title: "Sintaxe de padrões"
chapter_code: 19-03
slug: sintaxe-de-padroes
---

# Sintaxe de Padrões

Nesta seção, reunimos toda a sintaxe válida em padrões e discutimos por que e quando você pode querer usar cada uma.

## Casando com literais

Como você viu no Capítulo 6, é possível casar padrões diretamente com literais. O código a seguir dá alguns exemplos:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Este código imprime `one` porque o valor em `x` é `1`. Essa sintaxe é útil quando você quer que seu código tome uma ação se receber um valor concreto em particular.

## Casando com variáveis nomeadas

Variáveis nomeadas são padrões irrefutáveis que casam com qualquer valor, e já as usamos muitas vezes neste livro. Porém, há uma complicação quando você usa variáveis nomeadas em expressões `match`, `if let` ou `while let`. Como cada um desses tipos de expressão inicia um escopo novo, variáveis declaradas como parte de um padrão dentro dessas expressões farão shadow das que têm o mesmo nome fora das construções, como acontece com todas as variáveis. Na Listagem 19-11, declaramos uma variável chamada `x` com o valor `Some(5)` e uma variável `y` com o valor `10`. Em seguida, criamos uma expressão `match` sobre o valor `x`. Observe os padrões nos braços do `match` e o `println!` no final, e tente descobrir o que o código vai imprimir antes de executá-lo ou continuar lendo.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {x:?}"),
    }

    println!("at the end: x = {x:?}, y = {y}");
}
```

[Listagem 19-11](#listagem-19-11): Uma expressão `match` com um braço que introduz uma nova variável que faz shadow de uma variável `y` existente

Vamos percorrer o que acontece quando a expressão `match` é executada. O padrão no primeiro braço do `match` não casa com o valor definido de `x`, então o código continua.

O padrão no segundo braço do `match` introduz uma nova variável chamada `y` que casará com qualquer valor dentro de um `Some`. Como estamos em um escopo novo dentro da expressão `match`, esta é uma variável `y` nova, não a `y` que declaramos no início com o valor `10`. Esse novo binding de `y` casará com qualquer valor dentro de um `Some`, que é o que temos em `x`. Portanto, essa nova `y` se liga ao valor interno do `Some` em `x`. Esse valor é `5`, então a expressão daquele braço é executada e imprime `Matched, y = 5`.

Se `x` fosse um valor `None` em vez de `Some(5)`, os padrões dos dois primeiros braços não casariam, então o valor casaria com o underscore. Não introduzimos a variável `x` no padrão do braço com underscore, então o `x` na expressão ainda é o `x` externo que não foi alvo de shadow. Nesse caso hipotético, o `match` imprimiria `Default case, x = None`.

Quando a expressão `match` termina, seu escopo acaba, e o mesmo acontece com o escopo da `y` interna. O último `println!` produz `at the end: x = Some(5), y = 10`.

Para criar uma expressão `match` que compare os valores de `x` e `y` externos, em vez de introduzir uma nova variável que faz shadow da variável `y` existente, precisaríamos usar uma condicional match guard. Falaremos sobre match guards mais adiante na seção [“Adicionando condicionais com match guards”](#adicionando-condicionais-com-match-guards).

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

## Casando com vários padrões

Em expressões `match`, você pode casar com vários padrões usando a sintaxe `|`, que é o operador _ou_ de padrões. Por exemplo, no código a seguir, casamos o valor de `x` com os braços do `match`, sendo que o primeiro tem uma opção _ou_, o que significa que, se o valor de `x` casar com qualquer um dos valores daquele braço, o código desse braço será executado:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Este código imprime `one or two`.

## Casando intervalos de valores com `..=`

A sintaxe `..=` nos permite casar com um intervalo inclusivo de valores. No código a seguir, quando um padrão casa com qualquer um dos valores dentro do intervalo dado, aquele braço será executado:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
}
```

Se `x` for `1`, `2`, `3`, `4` ou `5`, o primeiro braço casará. Essa sintaxe é mais conveniente para vários valores de match do que usar o operador `|` para expressar a mesma ideia; se usássemos `|`, teríamos que especificar `1 | 2 | 3 | 4 | 5`. Especificar um intervalo é muito mais curto, especialmente se quisermos casar, digamos, qualquer número entre 1 e 1.000!

O compilador verifica em tempo de compilação se o intervalo não está vazio, e como os únicos tipos para os quais Rust consegue dizer se um intervalo está vazio ou não são `char` e valores numéricos, intervalos só são permitidos com valores numéricos ou `char`.

Aqui está um exemplo usando intervalos de valores `char`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 'c';

    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
}
```

Rust consegue dizer que `'c'` está dentro do intervalo do primeiro padrão e imprime `early ASCII letter`.

## Desestruturando para separar valores

Também podemos usar padrões para desestruturar structs, enums e tuplas e usar partes diferentes desses valores. Vamos percorrer cada tipo de valor.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

### Structs

A Listagem 19-12 mostra uma struct `Point` com dois campos, `x` e `y`, que podemos separar usando um padrão com uma instrução `let`.

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

[Listagem 19-12](#listagem-19-12): Desestruturando os campos de uma struct em variáveis separadas

Este código cria as variáveis `a` e `b` que casam com os valores dos campos `x` e `y` da struct `p`. Este exemplo mostra que os nomes das variáveis no padrão não precisam coincidir com os nomes dos campos da struct. Porém, é comum fazer os nomes das variáveis coincidirem com os dos campos para facilitar lembrar de quais campos cada variável veio. Por causa desse uso comum, e porque escrever `let Point { x: x, y: y } = p;` contém muita duplicação, Rust tem uma forma abreviada para padrões que casam com campos de struct: você só precisa listar o nome do campo da struct, e as variáveis criadas a partir do padrão terão os mesmos nomes. A Listagem 19-13 se comporta da mesma forma que o código da Listagem 19-12, mas as variáveis criadas no padrão `let` são `x` e `y` em vez de `a` e `b`.

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

[Listagem 19-13](#listagem-19-13): Desestruturando campos de struct usando a forma abreviada de campos de struct

Este código cria as variáveis `x` e `y` que casam com os campos `x` e `y` da variável `p`. O resultado é que as variáveis `x` e `y` contêm os valores da struct `p`.

Também podemos desestruturar com valores literais como parte do padrão da struct, em vez de criar variáveis para todos os campos. Isso nos permite testar alguns dos campos quanto a valores particulares enquanto criamos variáveis para desestruturar os outros campos.

Na Listagem 19-14, temos uma expressão `match` que separa valores `Point` em três casos: pontos que ficam diretamente no eixo `x` (o que é verdade quando `y = 0`), no eixo `y` (`x = 0`) ou em nenhum dos eixos.

**Arquivo: src/main.rs**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

[Listagem 19-14](#listagem-19-14): Desestruturando e casando com valores literais em um padrão

O primeiro braço casará com qualquer ponto que fique no eixo `x` ao especificar que o campo `y` casa se seu valor for o literal `0`. O padrão ainda cria uma variável `x` que podemos usar no código desse braço.

Da mesma forma, o segundo braço casa com qualquer ponto no eixo `y` ao especificar que o campo `x` casa se seu valor for `0` e cria uma variável `y` para o valor do campo `y`. O terceiro braço não especifica literais, então casa com qualquer outro `Point` e cria variáveis para os campos `x` e `y`.

Neste exemplo, o valor `p` casa com o segundo braço porque `x` contém um `0`, então este código imprimirá `On the y axis at 7`.

Lembre-se de que uma expressão `match` para de verificar braços assim que encontra o primeiro padrão que casa, então mesmo que `Point { x: 0, y: 0 }` esteja no eixo `x` e no eixo `y`, este código imprimiria apenas `On the x axis at 0`.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

### Enums

Já desestruturamos enums neste livro (por exemplo, a Listagem 6-5 no Capítulo 6), mas ainda não discutimos explicitamente que o padrão para desestruturar um enum corresponde à forma como os dados armazenados dentro do enum são definidos. Como exemplo, na Listagem 19-15, usamos o enum `Message` da Listagem 6-2 e escrevemos um `match` com padrões que desestruturarão cada valor interno.

**Arquivo: src/main.rs**

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
    }
}
```

[Listagem 19-15](#listagem-19-15): Desestruturando variantes de enum que armazenam diferentes tipos de valores

Este código imprimirá `Change color to red 0, green 160, and blue 255`. Tente alterar o valor de `msg` para ver o código dos outros braços ser executado.

Para variantes de enum sem nenhum dado, como `Message::Quit`, não podemos desestruturar o valor mais além. Só podemos casar com o valor literal `Message::Quit`, e não há variáveis nesse padrão.

Para variantes de enum semelhantes a struct, como `Message::Move`, podemos usar um padrão semelhante ao que especificamos para casar com structs. Depois do nome da variante, colocamos chaves e então listamos os campos com variáveis para separarmos as partes e usá-las no código desse braço. Aqui usamos a forma abreviada como fizemos na Listagem 19-13.

Para variantes de enum semelhantes a tupla, como `Message::Write`, que armazena uma tupla com um elemento, e `Message::ChangeColor`, que armazena uma tupla com três elementos, o padrão é semelhante ao que especificamos para casar com tuplas. O número de variáveis no padrão deve coincidir com o número de elementos na variante com a qual estamos casando.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

### Structs e enums aninhados

Até aqui, nossos exemplos casaram com structs ou enums apenas um nível abaixo, mas o matching também funciona com itens aninhados! Por exemplo, podemos refatorar o código da Listagem 19-15 para suportar cores RGB e HSV na mensagem `ChangeColor`, como mostrado na Listagem 19-16.

**Arquivo: src/main.rs**

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}");
        }
        _ => (),
    }
}
```

[Listagem 19-16](#listagem-19-16): Casando com enums aninhados

O padrão do primeiro braço na expressão `match` casa com uma variante `Message::ChangeColor` que contém uma variante `Color::Rgb`; em seguida, o padrão se liga aos três valores internos `i32`. O padrão do segundo braço também casa com uma variante `Message::ChangeColor`, mas o enum interno casa com `Color::Hsv`. Podemos especificar essas condições complexas em uma expressão `match`, mesmo envolvendo dois enums.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

### Structs e tuplas

Podemos misturar, combinar e aninhar padrões de desestruturação de formas ainda mais complexas. O exemplo a seguir mostra uma desestruturação complicada em que aninhamos structs e tuplas dentro de uma tupla e desestruturamos todos os valores primitivos:

**Arquivo: src/main.rs**

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
    }

    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
}
```

Este código nos permite separar tipos complexos em suas partes componentes para que possamos usar separadamente os valores que nos interessam.

Desestruturar com padrões é uma forma conveniente de usar pedaços de valores, como o valor de cada campo em uma struct, separadamente uns dos outros.

## Ignorando valores em um padrão

Você viu que às vezes é útil ignorar valores em um padrão, como no último braço de um `match`, para obter um catch-all que na prática não faz nada, mas cobre todos os valores possíveis restantes. Há algumas formas de ignorar valores inteiros ou partes de valores em um padrão: usando o padrão `_` (que você já viu), usando o padrão `_` dentro de outro padrão, usando um nome que começa com underscore ou usando `..` para ignorar partes restantes de um valor. Vamos explorar como e por que usar cada um desses padrões.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

### Um valor inteiro com `_`

Usamos o underscore como padrão curinga que casará com qualquer valor, mas não se liga a ele. Isso é especialmente útil como último braço em uma expressão `match`, mas também podemos usá-lo em qualquer padrão, incluindo parâmetros de função, como mostrado na Listagem 19-17.

**Arquivo: src/main.rs**

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {y}");
}

fn main() {
    foo(3, 4);
}
```

[Listagem 19-17](#listagem-19-17): Usando `_` em uma assinatura de função

Este código ignorará completamente o valor `3` passado como primeiro argumento e imprimirá `This code only uses the y parameter: 4`.

Na maioria dos casos em que você não precisa mais de um parâmetro de função em particular, alteraria a assinatura para que não incluísse o parâmetro não usado. Ignorar um parâmetro de função pode ser especialmente útil quando, por exemplo, você está implementando uma trait e precisa de uma assinatura de tipo específica, mas o corpo da função na sua implementação não precisa de um dos parâmetros. Assim você evita receber um aviso do compilador sobre parâmetros de função não usados, como aconteceria se usasse um nome.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

### Partes de um valor com `_` aninhado

Também podemos usar `_` dentro de outro padrão para ignorar apenas parte de um valor, por exemplo, quando queremos testar apenas parte de um valor, mas não temos uso para as outras partes no código correspondente que queremos executar. A Listagem 19-18 mostra código responsável por gerenciar o valor de uma configuração. Os requisitos de negócio são que o usuário não deve poder sobrescrever uma personalização existente de uma configuração, mas pode desfazer a configuração e atribuir um valor se ela estiver atualmente sem valor definido.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }

    println!("setting is {setting_value:?}");
}
```

[Listagem 19-18](#listagem-19-18): Usando um underscore dentro de padrões que casam com variantes `Some` quando não precisamos usar o valor dentro do `Some`

Este código imprimirá `Can't overwrite an existing customized value` e depois `setting is Some(5)`. No primeiro braço do `match`, não precisamos casar com ou usar os valores dentro de nenhuma das variantes `Some`, mas precisamos testar o caso em que `setting_value` e `new_setting_value` são a variante `Some`. Nesse caso, imprimimos o motivo de não alterar `setting_value`, e ela não é alterada.

Em todos os outros casos (se `setting_value` ou `new_setting_value` for `None`), expressos pelo padrão `_` no segundo braço, queremos permitir que `new_setting_value` se torne `setting_value`.

Também podemos usar underscores em vários lugares dentro de um padrão para ignorar valores particulares. A Listagem 19-19 mostra um exemplo de ignorar o segundo e o quarto valores em uma tupla de cinco itens.

**Arquivo: src/main.rs**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}");
        }
    }
}
```

[Listagem 19-19](#listagem-19-19): Ignorando várias partes de uma tupla

Este código imprimirá `Some numbers: 2, 8, 32`, e os valores `4` e `16` serão ignorados.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

### Uma variável não usada cujo nome começa com `_`

Se você cria uma variável, mas não a usa em lugar nenhum, Rust normalmente emite um aviso, porque uma variável não usada pode ser um bug. Porém, às vezes é útil poder criar uma variável que você ainda não vai usar, como quando está prototipando ou apenas começando um projeto. Nessa situação, você pode dizer ao Rust para não avisar sobre a variável não usada começando o nome da variável com um underscore. Na Listagem 19-20, criamos duas variáveis não usadas, mas ao compilar este código, devemos receber aviso apenas sobre uma delas.

**Arquivo: src/main.rs**

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

[Listagem 19-20](#listagem-19-20): Começando o nome de uma variável com underscore para evitar avisos de variável não usada

Aqui, recebemos um aviso por não usar a variável `y`, mas não recebemos aviso por não usar `_x`.

Note que há uma diferença sutil entre usar apenas `_` e usar um nome que começa com underscore. A sintaxe `_x` ainda liga o valor à variável, enquanto `_` não se liga de forma alguma. Para mostrar um caso em que essa distinção importa, a Listagem 19-21 nos dará um erro.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let s = Some(String::from("Hello!"));

    if let Some(_s) = s {
        println!("found a string");
    }

    println!("{s:?}");
}
```

[Listagem 19-21](#listagem-19-21): Uma variável não usada cujo nome começa com underscore ainda liga o valor, o que pode tomar posse do valor

Receberemos um erro porque o valor `s` ainda será movido para `_s`, o que nos impede de usar `s` novamente. Porém, usar o underscore sozinho nunca se liga ao valor. A Listagem 19-22 compilará sem erros porque `s` não é movido para `_`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }

    println!("{s:?}");
}
```

[Listagem 19-22](#listagem-19-22): Usar um underscore não liga o valor

Este código funciona perfeitamente porque nunca ligamos `s` a nada; ele não é movido.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

### Partes restantes de um valor com `..`

Com valores que têm muitas partes, podemos usar a sintaxe `..` para usar partes específicas e ignorar o restante, evitando a necessidade de listar underscores para cada valor ignorado. O padrão `..` ignora quaisquer partes de um valor que não casamos explicitamente no restante do padrão. Na Listagem 19-23, temos uma struct `Point` que armazena uma coordenada no espaço tridimensional. Na expressão `match`, queremos operar apenas na coordenada `x` e ignorar os valores nos campos `y` e `z`.

**Arquivo: src/main.rs**

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }

    let origin = Point { x: 0, y: 0, z: 0 };

    match origin {
        Point { x, .. } => println!("x is {x}"),
    }
}
```

[Listagem 19-23](#listagem-19-23): Ignorando todos os campos de um `Point` exceto `x` usando `..`

Listamos o valor `x` e então incluímos apenas o padrão `..`. Isso é mais rápido do que ter que listar `y: _` e `z: _`, particularmente quando estamos trabalhando com structs que têm muitos campos em situações em que apenas um ou dois campos são relevantes.

A sintaxe `..` se expandirá para quantos valores forem necessários. A Listagem 19-24 mostra como usar `..` com uma tupla.

**Arquivo: src/main.rs**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

[Listagem 19-24](#listagem-19-24): Casando apenas com o primeiro e o último valores em uma tupla e ignorando todos os outros

Neste código, o primeiro e o último valores casam com `first` e `last`. O `..` casará e ignorará tudo no meio.

Porém, usar `..` deve ser inequívoco. Se não estiver claro quais valores são destinados ao matching e quais devem ser ignorados, Rust nos dará um erro. A Listagem 19-25 mostra um exemplo de uso ambíguo de `..`, então não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {second}")
        },
    }
}
```

[Listagem 19-25](#listagem-19-25): Uma tentativa de usar `..` de forma ambígua

Ao compilar este exemplo, recebemos este erro:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

É impossível para Rust determinar quantos valores na tupla ignorar antes de casar um valor com `second` e então quantos valores adicionais ignorar depois disso. Este código poderia significar que queremos ignorar `2`, ligar `second` a `4` e então ignorar `8`, `16` e `32`; ou que queremos ignorar `2` e `4`, ligar `second` a `8` e então ignorar `16` e `32`; e assim por diante. O nome da variável `second` não significa nada especial para Rust, então recebemos um erro de compilação porque usar `..` em dois lugares assim é ambíguo.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

## Adicionando condicionais com match guards

Um _match guard_ é uma condicional `if` adicional, especificada depois do padrão em um braço de `match`, que também deve ser verdadeira para aquele braço ser escolhido. Match guards são úteis para expressar ideias mais complexas do que um padrão sozinho permite. Note, porém, que eles só estão disponíveis em expressões `match`, não em `if let` ou `while let`.

A condição pode usar variáveis criadas no padrão. A Listagem 19-26 mostra um `match` em que o primeiro braço tem o padrão `Some(x)` e também tem um match guard `if x % 2 == 0` (que será `true` se o número for par).

**Arquivo: src/main.rs**

```rust
fn main() {
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {x} is even"),
        Some(x) => println!("The number {x} is odd"),
        None => (),
    }
}
```

[Listagem 19-26](#listagem-19-26): Adicionando um match guard a um padrão

Este exemplo imprimirá `The number 4 is even`. Quando `num` é comparado ao padrão no primeiro braço, ele casa porque `Some(4)` casa com `Some(x)`. Então, o match guard verifica se o resto da divisão de `x` por 2 é igual a 0, e como é, o primeiro braço é selecionado.

Se `num` fosse `Some(5)` em vez disso, o match guard no primeiro braço seria `false` porque o resto de 5 dividido por 2 é 1, que não é igual a 0. Rust então iria para o segundo braço, que casaria porque o segundo braço não tem match guard e, portanto, casa com qualquer variante `Some`.

Não há como expressar a condição `if x % 2 == 0` dentro de um padrão, então o match guard nos dá a capacidade de expressar essa lógica. A desvantagem dessa expressividade adicional é que o compilador não tenta verificar exaustividade quando expressões match guard estão envolvidas.

Ao discutir a Listagem 19-11, mencionamos que poderíamos usar match guards para resolver nosso problema de shadow de padrão. Lembre-se de que criamos uma nova variável dentro do padrão na expressão `match` em vez de usar a variável fora do `match`. Essa nova variável significava que não podíamos testar contra o valor da variável externa. A Listagem 19-27 mostra como podemos usar um match guard para corrigir esse problema.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {x:?}"),
    }

    println!("at the end: x = {x:?}, y = {y}");
}
```

[Listagem 19-27](#listagem-19-27): Usando um match guard para testar igualdade com uma variável externa

Este código agora imprimirá `Default case, x = Some(5)`. O padrão no segundo braço do `match` não introduz uma nova variável `y` que faria shadow da `y` externa, o que significa que podemos usar a `y` externa no match guard. Em vez de especificar o padrão como `Some(y)`, o que faria shadow da `y` externa, especificamos `Some(n)`. Isso cria uma nova variável `n` que não faz shadow de nada porque não há variável `n` fora do `match`.

O match guard `if n == y` não é um padrão e, portanto, não introduz novas variáveis. Essa `y` _é_ a `y` externa, em vez de uma nova `y` fazendo shadow dela, e podemos procurar um valor que tenha o mesmo valor da `y` externa comparando `n` com `y`.

Você também pode usar o operador _ou_ `|` em um match guard para especificar vários padrões; a condição do match guard se aplicará a todos os padrões. A Listagem 19-28 mostra a precedência ao combinar um padrão que usa `|` com um match guard. A parte importante deste exemplo é que o match guard `if y` se aplica a `4`, `5` _e_ `6`, mesmo que possa parecer que `if y` se aplica apenas a `6`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
}
```

[Listagem 19-28](#listagem-19-28): Combinando vários padrões com um match guard

A condição do match afirma que o braço só casa se o valor de `x` for igual a `4`, `5` ou `6` _e_ se `y` for `true`. Quando este código é executado, o padrão do primeiro braço casa porque `x` é `4`, mas o match guard `if y` é `false`, então o primeiro braço não é escolhido. O código passa para o segundo braço, que casa, e este programa imprime `no`. O motivo é que a condição `if` se aplica ao padrão inteiro `4 | 5 | 6`, não apenas ao último valor `6`. Em outras palavras, a precedência de um match guard em relação a um padrão se comporta assim:

```text
(4 | 5 | 6) if y => ...
```

em vez de:

```text
4 | 5 | (6 if y) => ...
```

Depois de executar o código, o comportamento de precedência fica evidente: se o match guard se aplicasse apenas ao valor final na lista de valores especificada com o operador `|`, o braço teria casado e o programa teria impresso `yes`.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

## Usando bindings `@`

O operador _at_ `@` nos permite criar uma variável que armazena um valor ao mesmo tempo em que testamos esse valor quanto a um padrão. Na Listagem 19-29, queremos testar se o campo `id` de um `Message::Hello` está dentro do intervalo `3..=7`. Também queremos ligar o valor à variável `id` para que possamos usá-lo no código associado ao braço.

**Arquivo: src/main.rs**

```rust
fn main() {
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello { id: id @ 3..=7 } => {
            println!("Found an id in range: {id}")
        }
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {id}"),
    }
}
```

[Listagem 19-29](#listagem-19-29): Usando `@` para ligar a um valor em um padrão enquanto também o testa

Este exemplo imprimirá `Found an id in range: 5`. Ao especificar `id @` antes do intervalo `3..=7`, estamos capturando qualquer valor que casou com o intervalo em uma variável chamada `id` enquanto também testamos se o valor casou com o padrão de intervalo.

No segundo braço, onde temos apenas um intervalo especificado no padrão, o código associado ao braço não tem uma variável que contenha o valor real do campo `id`. O valor do campo `id` poderia ter sido 10, 11 ou 12, mas o código que acompanha esse padrão não sabe qual é. O código do padrão não consegue usar o valor do campo `id` porque não salvamos o valor de `id` em uma variável.

No último braço, onde especificamos uma variável sem intervalo, temos o valor disponível para usar no código do braço em uma variável chamada `id`. O motivo é que usamos a sintaxe abreviada de campos de struct. Mas não aplicamos nenhum teste ao valor no campo `id` neste braço, como fizemos nos dois primeiros braços: qualquer valor casaria com este padrão.

Usar `@` nos permite testar um valor e salvá-lo em uma variável dentro de um padrão.

## Resumo

Os padrões de Rust são muito úteis para distinguir entre diferentes tipos de dados. Quando usados em expressões `match`, Rust garante que seus padrões cubram todos os valores possíveis, ou seu programa não compilará. Padrões em instruções `let` e parâmetros de função tornam essas construções mais úteis, permitindo desestruturar valores em partes menores e atribuir essas partes a variáveis. Podemos criar padrões simples ou complexos conforme nossas necessidades.

Em seguida, no penúltimo capítulo do livro, veremos alguns aspectos avançados de vários recursos do Rust.
