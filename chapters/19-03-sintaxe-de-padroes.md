---
title: "Sintaxe de padrões"
chapter_code: 19-03
slug: sintaxe-de-padroes
---

# Sintaxe de padrões

Nesta seção, reunimos toda a sintaxe que pode aparecer em padrões e discutimos por que, e quando, você talvez queira usar cada forma.

## Casando com literais

Como você viu no Capítulo 6, é possível fazer padrões casarem diretamente com literais. O código a seguir mostra alguns exemplos:

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

Esse código imprime `one` porque o valor em `x` é `1`. Essa sintaxe é útil quando você quer que o código execute uma ação ao receber um valor concreto específico.

## Casando com variáveis nomeadas

Variáveis nomeadas são padrões irrefutáveis que casam com qualquer valor, e já as usamos muitas vezes neste livro. Porém, há uma complicação quando usamos variáveis nomeadas em expressões `match`, `if let` ou `while let`. Como cada uma dessas expressões inicia um novo escopo, variáveis declaradas como parte de um padrão dentro delas sombreiam variáveis de mesmo nome fora da construção, como acontece com todas as variáveis. Na Listagem 19-11, declaramos uma variável chamada `x` com o valor `Some(5)` e uma variável `y` com o valor `10`. Em seguida, criamos uma expressão `match` sobre o valor `x`. Observe os padrões nos braços do `match` e o `println!` final, e tente descobrir o que o código vai imprimir antes de executá-lo ou continuar lendo.

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

<a id="listagem-19-11"></a>

[Listagem 19-11](#listagem-19-11): Uma expressão `match` com um braço que introduz uma nova variável que faz shadow de uma variável `y` existente

Vamos percorrer o que acontece quando a expressão `match` é executada. O padrão no primeiro braço do `match` não casa com o valor definido em `x`, então a execução continua para o próximo braço.

O padrão no segundo braço do `match` introduz uma nova variável chamada `y`, que casa com qualquer valor dentro de um `Some`. Como estamos em um novo escopo dentro da expressão `match`, essa é uma nova variável `y`, não a `y` que declaramos no início com o valor `10`. Esse novo binding de `y` casa com qualquer valor dentro de um `Some`, que é justamente o que temos em `x`. Portanto, essa nova `y` se vincula ao valor interno do `Some` em `x`. Esse valor é `5`, então a expressão daquele braço é executada e imprime `Matched, y = 5`.

Se `x` fosse `None` em vez de `Some(5)`, os padrões dos dois primeiros braços não casariam, então o valor casaria com o underscore. Não introduzimos uma variável `x` no padrão do braço com underscore, então o `x` na expressão ainda é o `x` externo, que não foi sombreado. Nesse caso hipotético, o `match` imprimiria `Default case, x = None`.

Quando a expressão `match` termina, seu escopo acaba, e o escopo da `y` interna acaba junto. O último `println!` produz `at the end: x = Some(5), y = 10`.

Para criar uma expressão `match` que compare os valores externos de `x` e `y`, em vez de introduzir uma nova variável que sombreia a variável `y` existente, precisaríamos usar uma condicional chamada match guard. Falaremos sobre match guards mais adiante na seção [“Adicionando condicionais com match guards”](#adicionando-condicionais-com-match-guards).

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

## Casando com vários padrões

Em expressões `match`, você pode casar com vários padrões usando a sintaxe `|`, que é o operador _ou_ de padrões. Por exemplo, no código a seguir, comparamos o valor de `x` com os braços do `match`; o primeiro braço tem uma opção _ou_, o que significa que, se o valor de `x` casar com qualquer um dos valores daquele braço, o código desse braço será executado:

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

Se `x` for `1`, `2`, `3`, `4` ou `5`, o primeiro braço casará. Essa sintaxe é mais conveniente para vários valores de match do que usar o operador `|` para expressar a mesma ideia; se usássemos `|`, teríamos que escrever `1 | 2 | 3 | 4 | 5`. Especificar um intervalo é muito mais curto, especialmente se quisermos casar, digamos, qualquer número entre 1 e 1.000!

O compilador verifica em tempo de compilação se o intervalo não está vazio. Como os únicos tipos para os quais Rust consegue determinar se um intervalo está vazio são `char` e valores numéricos, intervalos só são permitidos com valores numéricos ou `char`.

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

Também podemos usar padrões para desestruturar structs, enums e tuplas e trabalhar com partes diferentes desses valores. Vamos passar por cada tipo.

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

<a id="listagem-19-12"></a>

[Listagem 19-12](#listagem-19-12): Desestruturando os campos de uma struct em variáveis separadas

Esse código cria as variáveis `a` e `b`, que casam com os valores dos campos `x` e `y` da struct `p`. O exemplo mostra que os nomes das variáveis no padrão não precisam coincidir com os nomes dos campos da struct. Porém, é comum usar os mesmos nomes dos campos para facilitar lembrar de onde cada variável veio. Por causa desse uso comum, e porque escrever `let Point { x: x, y: y } = p;` contém muita repetição, Rust oferece uma forma abreviada para padrões que casam com campos de struct: basta listar o nome do campo, e as variáveis criadas pelo padrão terão os mesmos nomes. A Listagem 19-13 se comporta da mesma forma que a Listagem 19-12, mas as variáveis criadas no padrão `let` são `x` e `y` em vez de `a` e `b`.

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

<a id="listagem-19-13"></a>

[Listagem 19-13](#listagem-19-13): Desestruturando campos de struct usando a forma abreviada de campos de struct

Este código cria as variáveis `x` e `y` que casam com os campos `x` e `y` da variável `p`. O resultado é que as variáveis `x` e `y` contêm os valores da struct `p`.

Também podemos desestruturar usando valores literais como parte do padrão da struct, em vez de criar variáveis para todos os campos. Isso nos permite testar alguns campos contra valores específicos enquanto criamos variáveis para desestruturar os outros.

Na Listagem 19-14, temos uma expressão `match` que separa valores `Point` em três casos: pontos que ficam diretamente no eixo `x` (quando `y = 0`), pontos que ficam no eixo `y` (quando `x = 0`) ou pontos que não ficam em nenhum dos eixos.

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

<a id="listagem-19-14"></a>

[Listagem 19-14](#listagem-19-14): Desestruturando e casando com valores literais em um padrão

O primeiro braço casará com qualquer ponto que fique no eixo `x` ao especificar que o campo `y` casa se seu valor for o literal `0`. O padrão ainda cria uma variável `x` que podemos usar no código desse braço.

Da mesma forma, o segundo braço casa com qualquer ponto no eixo `y` ao especificar que o campo `x` casa se seu valor for `0` e cria uma variável `y` para o valor do campo `y`. O terceiro braço não especifica literais, então casa com qualquer outro `Point` e cria variáveis para os campos `x` e `y`.

Neste exemplo, o valor `p` casa com o segundo braço porque `x` contém um `0`, então este código imprimirá `On the y axis at 7`.

Lembre-se de que uma expressão `match` para de verificar braços assim que encontra o primeiro padrão que casa. Portanto, mesmo que `Point { x: 0, y: 0 }` esteja tanto no eixo `x` quanto no eixo `y`, esse código imprimiria apenas `On the x axis at 0`.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

### Enums

Já desestruturamos enums neste livro, por exemplo na Listagem 6-5 do Capítulo 6, mas ainda não discutimos explicitamente que o padrão usado para desestruturar um enum corresponde à forma como os dados armazenados dentro dele foram definidos. Como exemplo, na Listagem 19-15, usamos o enum `Message` da Listagem 6-2 e escrevemos um `match` com padrões que desestruturam cada valor interno.

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

<a id="listagem-19-15"></a>

[Listagem 19-15](#listagem-19-15): Desestruturando variantes de enum que armazenam diferentes tipos de valores

Este código imprimirá `Change color to red 0, green 160, and blue 255`. Tente alterar o valor de `msg` para ver o código dos outros braços ser executado.

Para variantes de enum sem nenhum dado, como `Message::Quit`, não podemos desestruturar o valor mais além. Só podemos casar com o valor literal `Message::Quit`, e não há variáveis nesse padrão.

Para variantes de enum semelhantes a struct, como `Message::Move`, podemos usar um padrão semelhante ao que especificamos para casar com structs. Depois do nome da variante, colocamos chaves e então listamos os campos com variáveis para separarmos as partes e usá-las no código desse braço. Aqui usamos a forma abreviada como fizemos na Listagem 19-13.

Para variantes de enum semelhantes a tupla, como `Message::Write`, que armazena uma tupla com um elemento, e `Message::ChangeColor`, que armazena uma tupla com três elementos, o padrão é semelhante ao que especificamos para casar com tuplas. O número de variáveis no padrão deve coincidir com o número de elementos na variante com a qual estamos casando.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

### Structs e enums aninhados

Até aqui, nossos exemplos casaram com structs ou enums apenas um nível abaixo, mas matching também funciona com itens aninhados! Por exemplo, podemos refatorar o código da Listagem 19-15 para dar suporte a cores RGB e HSV na mensagem `ChangeColor`, como mostra a Listagem 19-16.

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

<a id="listagem-19-16"></a>

[Listagem 19-16](#listagem-19-16): Casando com enums aninhados

O padrão do primeiro braço na expressão `match` casa com uma variante `Message::ChangeColor` que contém uma variante `Color::Rgb`; em seguida, o padrão se liga aos três valores internos `i32`. O padrão do segundo braço também casa com uma variante `Message::ChangeColor`, mas o enum interno casa com `Color::Hsv`. Podemos especificar essas condições complexas em uma expressão `match`, mesmo envolvendo dois enums.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

### Structs e tuplas

Podemos misturar, combinar e aninhar padrões de desestruturação de formas ainda mais complexas. O exemplo a seguir mostra uma desestruturação mais elaborada, em que aninhamos structs e tuplas dentro de uma tupla e desestruturamos todos os valores primitivos:

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

Esse código nos permite separar tipos complexos em suas partes componentes para usar separadamente os valores que nos interessam.

Desestruturar com padrões é uma forma conveniente de trabalhar com pedaços de valores, como cada campo de uma struct, separadamente.

## Ignorando valores em um padrão

Você viu que às vezes é útil ignorar valores em um padrão, como no último braço de um `match`, para ter um pega-tudo que na prática não faz nada, mas cobre todos os valores restantes possíveis. Há algumas formas de ignorar valores inteiros ou partes de valores em um padrão: usando o padrão `_`, que você já viu; usando `_` dentro de outro padrão; usando um nome que começa com underscore; ou usando `..` para ignorar as partes restantes de um valor. Vamos explorar como e por que usar cada uma dessas formas.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

### Um valor inteiro com `_`

Usamos o underscore como um padrão curinga que casa com qualquer valor, mas não se vincula a ele. Isso é especialmente útil como último braço em uma expressão `match`, mas também podemos usá-lo em qualquer padrão, incluindo parâmetros de função, como mostra a Listagem 19-17.

**Arquivo: src/main.rs**

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {y}");
}

fn main() {
    foo(3, 4);
}
```

<a id="listagem-19-17"></a>

[Listagem 19-17](#listagem-19-17): Usando `_` em uma assinatura de função

Este código ignorará completamente o valor `3` passado como primeiro argumento e imprimirá `This code only uses the y parameter: 4`.

Na maioria dos casos em que você não precisa mais de um parâmetro de função específico, alteraria a assinatura para removê-lo. Ignorar um parâmetro de função pode ser especialmente útil quando, por exemplo, você está implementando uma trait e precisa seguir uma assinatura específica, mas o corpo da função na sua implementação não usa um dos parâmetros. Assim, você evita receber um aviso do compilador sobre parâmetros de função não usados, como aconteceria se usasse um nome comum.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

### Partes de um valor com `_` aninhado

Também podemos usar `_` dentro de outro padrão para ignorar apenas parte de um valor. Isso é útil, por exemplo, quando queremos testar uma parte de um valor, mas não precisamos das outras partes no código que será executado. A Listagem 19-18 mostra um código responsável por gerenciar o valor de uma configuração. Os requisitos de negócio dizem que o usuário não deve poder sobrescrever uma personalização existente, mas pode desfazer a configuração e atribuir um valor se ela estiver atualmente sem valor definido.

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

<a id="listagem-19-18"></a>

[Listagem 19-18](#listagem-19-18): Usando um underscore dentro de padrões que casam com variantes `Some` quando não precisamos usar o valor dentro do `Some`

Esse código imprimirá `Can't overwrite an existing customized value` e depois `setting is Some(5)`. No primeiro braço do `match`, não precisamos casar com os valores internos das variantes `Some`, nem usá-los, mas precisamos testar o caso em que `setting_value` e `new_setting_value` são ambas variantes `Some`. Nesse caso, imprimimos o motivo de não alterar `setting_value`, e ela permanece como estava.

Em todos os outros casos, isto é, se `setting_value` ou `new_setting_value` for `None`, expressos pelo padrão `_` no segundo braço, queremos permitir que `new_setting_value` se torne `setting_value`.

Também podemos usar underscores em vários lugares dentro de um padrão para ignorar valores específicos. A Listagem 19-19 mostra um exemplo que ignora o segundo e o quarto valores em uma tupla de cinco itens.

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

<a id="listagem-19-19"></a>

[Listagem 19-19](#listagem-19-19): Ignorando várias partes de uma tupla

Este código imprimirá `Some numbers: 2, 8, 32`, e os valores `4` e `16` serão ignorados.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

### Uma variável não usada cujo nome começa com `_`

Se você cria uma variável, mas não a usa em lugar nenhum, Rust normalmente emite um aviso, porque uma variável não usada pode indicar um bug. Às vezes, porém, é útil criar uma variável que você ainda não vai usar, como ao prototipar ou iniciar um projeto. Nessa situação, você pode dizer ao Rust para não avisar sobre a variável não usada começando o nome dela com um underscore. Na Listagem 19-20, criamos duas variáveis não usadas, mas, ao compilar esse código, devemos receber aviso sobre apenas uma delas.

**Arquivo: src/main.rs**

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

<a id="listagem-19-20"></a>

[Listagem 19-20](#listagem-19-20): Começando o nome de uma variável com underscore para evitar avisos de variável não usada

Aqui, recebemos um aviso por não usar a variável `y`, mas não recebemos aviso por não usar `_x`.

Note que há uma diferença sutil entre usar apenas `_` e usar um nome que começa com underscore. A sintaxe `_x` ainda vincula o valor à variável, enquanto `_` não se vincula a nada. Para mostrar um caso em que essa distinção importa, a Listagem 19-21 produzirá um erro.

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

<a id="listagem-19-21"></a>

[Listagem 19-21](#listagem-19-21): Uma variável não usada cujo nome começa com underscore ainda liga o valor, o que pode tomar posse do valor

Receberemos um erro porque o valor `s` ainda será movido para `_s`, o que nos impede de usar `s` novamente. Porém, usar apenas o underscore nunca se vincula ao valor. A Listagem 19-22 compilará sem erros porque `s` não é movido para `_`.

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

<a id="listagem-19-22"></a>

[Listagem 19-22](#listagem-19-22): Usar um underscore não liga o valor

Este código funciona perfeitamente porque nunca ligamos `s` a nada; ele não é movido.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

### Partes restantes de um valor com `..`

Com valores que têm muitas partes, podemos usar a sintaxe `..` para aproveitar partes específicas e ignorar o restante, evitando listar underscores para cada valor ignorado. O padrão `..` ignora quaisquer partes de um valor que não casamos explicitamente no restante do padrão. Na Listagem 19-23, temos uma struct `Point` que armazena uma coordenada no espaço tridimensional. Na expressão `match`, queremos operar apenas sobre a coordenada `x` e ignorar os valores dos campos `y` e `z`.

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

<a id="listagem-19-23"></a>

[Listagem 19-23](#listagem-19-23): Ignorando todos os campos de um `Point` exceto `x` usando `..`

Listamos o valor `x` e então incluímos o padrão `..`. Isso é mais rápido do que escrever `y: _` e `z: _`, especialmente quando trabalhamos com structs que têm muitos campos e apenas um ou dois são relevantes.

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

<a id="listagem-19-24"></a>

[Listagem 19-24](#listagem-19-24): Casando apenas com o primeiro e o último valores em uma tupla e ignorando todos os outros

Neste código, o primeiro e o último valores casam com `first` e `last`. O `..` casará e ignorará tudo no meio.

Porém, o uso de `..` precisa ser inequívoco. Se não estiver claro quais valores devem participar do matching e quais devem ser ignorados, Rust emitirá um erro. A Listagem 19-25 mostra um exemplo ambíguo de `..`, que portanto não compila.

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

<a id="listagem-19-25"></a>

[Listagem 19-25](#listagem-19-25): Uma tentativa de usar `..` de forma ambígua

Ao compilar este exemplo, recebemos este erro:

```console
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

É impossível para Rust determinar quantos valores da tupla deve ignorar antes de casar um valor com `second` e quantos valores adicionais deve ignorar depois disso. Esse código poderia significar que queremos ignorar `2`, vincular `second` a `4` e depois ignorar `8`, `16` e `32`; ou que queremos ignorar `2` e `4`, vincular `second` a `8` e depois ignorar `16` e `32`; e assim por diante. O nome da variável `second` não tem nenhum significado especial para Rust, então recebemos um erro de compilação porque usar `..` em dois lugares assim é ambíguo.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

## Adicionando condicionais com match guards

Um _match guard_ é uma condicional `if` adicional, escrita depois do padrão em um braço de `match`, que também precisa ser verdadeira para que aquele braço seja escolhido. Match guards são úteis para expressar ideias mais complexas do que um padrão sozinho permite. Note, porém, que eles só estão disponíveis em expressões `match`, não em `if let` nem em `while let`.

A condição pode usar variáveis criadas no padrão. A Listagem 19-26 mostra um `match` em que o primeiro braço tem o padrão `Some(x)` e também um match guard `if x % 2 == 0`, que será `true` se o número for par.

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

<a id="listagem-19-26"></a>

[Listagem 19-26](#listagem-19-26): Adicionando um match guard a um padrão

Esse exemplo imprimirá `The number 4 is even`. Quando `num` é comparado ao padrão no primeiro braço, ele casa porque `Some(4)` casa com `Some(x)`. Então, o match guard verifica se o resto da divisão de `x` por 2 é igual a 0; como é, o primeiro braço é selecionado.

Se `num` fosse `Some(5)`, o match guard no primeiro braço seria `false`, porque o resto de 5 dividido por 2 é 1, não 0. Rust então seguiria para o segundo braço, que casaria porque não tem match guard e, portanto, casa com qualquer variante `Some`.

Não há como expressar a condição `if x % 2 == 0` dentro de um padrão, então o match guard nos dá uma forma de expressar essa lógica. A desvantagem dessa expressividade adicional é que o compilador não tenta verificar exaustividade quando match guards estão envolvidos.

Ao discutir a Listagem 19-11, mencionamos que poderíamos usar match guards para resolver nosso problema de shadowing no padrão. Lembre-se de que criamos uma nova variável dentro do padrão da expressão `match` em vez de usar a variável de fora do `match`. Essa nova variável impedia que testássemos contra o valor da variável externa. A Listagem 19-27 mostra como usar um match guard para corrigir esse problema.

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

<a id="listagem-19-27"></a>

[Listagem 19-27](#listagem-19-27): Usando um match guard para testar igualdade com uma variável externa

Esse código agora imprimirá `Default case, x = Some(5)`. O padrão no segundo braço do `match` não introduz uma nova variável `y` que sombrearia a `y` externa, o que significa que podemos usar a `y` externa no match guard. Em vez de especificar o padrão como `Some(y)`, o que sombrearia a `y` externa, especificamos `Some(n)`. Isso cria uma nova variável `n` que não sombreia nada, porque não há variável `n` fora do `match`.

O match guard `if n == y` não é um padrão e, portanto, não introduz novas variáveis. Essa `y` _é_ a `y` externa, não uma nova `y` sombreando a anterior, e podemos procurar um valor igual ao da `y` externa comparando `n` com `y`.

Também podemos usar o operador _ou_ `|` junto com um match guard para especificar vários padrões; a condição do match guard se aplica a todos eles. A Listagem 19-28 mostra a precedência ao combinar um padrão que usa `|` com um match guard. O ponto importante deste exemplo é que o match guard `if y` se aplica a `4`, `5` _e_ `6`, mesmo que possa parecer que `if y` se aplica apenas a `6`.

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

<a id="listagem-19-28"></a>

[Listagem 19-28](#listagem-19-28): Combinando vários padrões com um match guard

A condição do match diz que o braço só casa se o valor de `x` for `4`, `5` ou `6` _e_ se `y` for `true`. Quando esse código é executado, o padrão do primeiro braço casa porque `x` é `4`, mas o match guard `if y` é `false`, então o primeiro braço não é escolhido. O código passa para o segundo braço, que casa, e o programa imprime `no`. O motivo é que a condição `if` se aplica ao padrão inteiro `4 | 5 | 6`, não apenas ao último valor `6`. Em outras palavras, a precedência de um match guard em relação a um padrão se comporta assim:

```text
(4 | 5 | 6) if y => ...
```

em vez de:

```text
4 | 5 | (6 if y) => ...
```

Depois de executar o código, o comportamento de precedência fica evidente: se o match guard se aplicasse apenas ao valor final da lista especificada com o operador `|`, o braço teria casado e o programa teria impresso `yes`.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

## Usando bindings `@`

O operador _at_ `@` nos permite criar uma variável que armazena um valor ao mesmo tempo que testamos esse valor contra um padrão. Na Listagem 19-29, queremos testar se o campo `id` de um `Message::Hello` está dentro do intervalo `3..=7`. Também queremos vincular o valor à variável `id`, para que possamos usá-lo no código associado ao braço.

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

<a id="listagem-19-29"></a>

[Listagem 19-29](#listagem-19-29): Usando `@` para ligar a um valor em um padrão enquanto também o testa

Esse exemplo imprimirá `Found an id in range: 5`. Ao especificar `id @` antes do intervalo `3..=7`, capturamos qualquer valor que casou com o intervalo em uma variável chamada `id`, enquanto também testamos se o valor casou com o padrão de intervalo.

No segundo braço, em que temos apenas um intervalo especificado no padrão, o código associado ao braço não tem uma variável que contenha o valor real do campo `id`. O valor poderia ser 10, 11 ou 12, mas o código daquele padrão não sabe qual deles é. O código do braço não consegue usar o valor do campo `id` porque não salvamos esse valor em uma variável.

No último braço, em que especificamos uma variável sem intervalo, temos o valor disponível para usar no código do braço em uma variável chamada `id`. Isso acontece porque usamos a sintaxe abreviada de campos de struct. Mas, nesse braço, não aplicamos nenhum teste ao valor do campo `id`, como fizemos nos dois primeiros braços: qualquer valor casaria com esse padrão.

Usar `@` nos permite testar um valor e salvá-lo em uma variável dentro do mesmo padrão.

## Resumo

Os padrões de Rust são muito úteis para distinguir entre diferentes formatos de dados. Quando usados em expressões `match`, Rust garante que seus padrões cubram todos os valores possíveis, ou o programa não compilará. Padrões em instruções `let` e parâmetros de função tornam essas construções mais úteis, permitindo desestruturar valores em partes menores e atribuir essas partes a variáveis. Podemos criar padrões simples ou complexos conforme a necessidade.

Em seguida, no penúltimo capítulo do livro, veremos alguns aspectos avançados de vários recursos do Rust.
