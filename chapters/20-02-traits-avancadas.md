---
title: "Traits avançadas"
chapter_code: 20-02
slug: traits-avancadas
---

# Traits Avançadas

Vimos traits pela primeira vez na seção Definindo comportamento compartilhado com traits do Capítulo 10, mas não entramos nos detalhes mais avançados. Agora que você sabe mais sobre Rust, podemos ir ao que interessa.

## Definindo traits com tipos associados

_Tipos associados_ conectam um placeholder de tipo a uma trait de modo que as definições de métodos da trait possam usar esses placeholders nas assinaturas. Quem implementa a trait especifica o tipo concreto no lugar do placeholder para aquela implementação. Assim, definimos uma trait que usa alguns tipos sem precisar saber exatamente quais são até a implementação.

Descrevemos a maior parte dos recursos avançados deste capítulo como raramente necessários. Tipos associados ficam no meio: são usados com menos frequência que os recursos do resto do livro, mas mais que muitos outros deste capítulo.

Um exemplo de trait com tipo associado é a trait `Iterator` da biblioteca padrão. O tipo associado se chama `Item` e representa o tipo dos valores sobre os quais quem implementa `Iterator` itera. A definição de `Iterator` aparece na Listagem 20-13.

**Arquivo: src/lib.rs**

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

<a id="listagem-20-13"></a>

[Listagem 20-13](#listagem-20-13): Definição da trait `Iterator` com o tipo associado `Item`

O tipo `Item` é um placeholder, e a definição de `next` mostra que retornará valores do tipo `Option<Self::Item>`. Quem implementa `Iterator` especifica o tipo concreto de `Item`, e `next` retorna um `Option` com valor desse tipo concreto.

Tipos associados podem parecer parecidos com generics, que permitem definir uma função sem especificar quais tipos ela aceita. Para ver a diferença, vejamos uma implementação de `Iterator` em um tipo `Counter` que define `Item` como `u32`:

**Arquivo: src/lib.rs**

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}
```

Essa sintaxe lembra a de generics. Então por que não definir `Iterator` com generics, como na Listagem 20-14?

**Arquivo: src/lib.rs**

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

<a id="listagem-20-14"></a>

[Listagem 20-14](#listagem-20-14): Definição hipotética de `Iterator` usando generics

A diferença é que, com generics como na Listagem 20-14, precisamos anotar os tipos em cada implementação; como também podemos implementar `Iterator<String> for Counter` ou qualquer outro tipo, podemos ter várias implementações de `Iterator` para `Counter`. Ou seja, quando uma trait tem parâmetro genérico, ela pode ser implementada várias vezes para o mesmo tipo, mudando os tipos concretos dos parâmetros genéricos a cada vez. Ao usar `next` em `Counter`, teríamos de fornecer anotações de tipo indicando qual implementação de `Iterator` queremos.

Com tipos associados, não precisamos anotar tipos, porque não podemos implementar uma trait várias vezes no mesmo tipo. Na Listagem 20-13, com a definição que usa tipos associados, escolhemos o tipo de `Item` apenas uma vez, pois só pode haver um `impl Iterator for Counter`. Não precisamos especificar que queremos um iterador de valores `u32` em todo lugar em que chamamos `next` em `Counter`.

Tipos associados também fazem parte do contrato da trait: quem implementa deve fornecer um tipo para o placeholder do tipo associado. Tipos associados costumam ter nome que descreve o uso do tipo, e documentá-los na API é uma boa prática.

## Usando parâmetros de tipo genérico padrão e sobrecarga de operadores

Ao usar parâmetros de tipo genérico, podemos especificar um tipo concreto padrão para o genérico. Isso elimina a necessidade de quem implementa a trait especificar um tipo concreto quando o padrão serve. Você indica o tipo padrão ao declarar um tipo genérico com a sintaxe `<PlaceholderType=ConcreteType>`.

Um ótimo exemplo é a _sobrecarga de operadores_, em que você personaliza o comportamento de um operador (como `+`) em situações específicas.

Rust não permite criar operadores próprios nem sobrecarregar operadores arbitrários. Mas você pode sobrecarregar as operações e traits correspondentes listadas em `std::ops` implementando as traits associadas ao operador. Por exemplo, na Listagem 20-15 sobrecarregamos o operador `+` para somar duas instâncias de `Point`. Fazemos isso implementando a trait `Add` em uma struct `Point`.

**Arquivo: src/main.rs**

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

<a id="listagem-20-15"></a>

[Listagem 20-15](#listagem-20-15): Implementando a trait `Add` para sobrecarregar o operador `+` em instâncias de `Point`

O método `add` soma os valores `x` de duas instâncias de `Point` e os valores `y` de duas instâncias de `Point` para criar um novo `Point`. A trait `Add` tem um tipo associado chamado `Output` que determina o tipo retornado por `add`.

O tipo genérico padrão neste código está na trait `Add`. Eis a definição:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Este código deve parecer familiar: uma trait com um método e um tipo associado. A parte nova é `Rhs=Self`: essa sintaxe chama-se _parâmetros de tipo padrão_. O parâmetro de tipo genérico `Rhs` (abreviação de “right-hand side”, lado direito) define o tipo do parâmetro `rhs` em `add`. Se não especificarmos um tipo concreto para `Rhs` ao implementar `Add`, o tipo de `Rhs` será `Self` por padrão — o tipo em que estamos implementando `Add`.

Quando implementamos `Add` para `Point`, usamos o padrão de `Rhs` porque queríamos somar duas instâncias de `Point`. Vejamos um exemplo em que personalizamos o tipo `Rhs` em vez do padrão.

Temos duas structs, `Millimeters` e `Meters`, guardando valores em unidades diferentes. Esse envolvimento fino de um tipo existente em outra struct é o _padrão newtype_, descrito com mais detalhe em Implementando traits externas com o padrão newtype. Queremos somar valores em milímetros a valores em metros e que a implementação de `Add` faça a conversão corretamente. Podemos implementar `Add` para `Millimeters` com `Meters` como `Rhs`, como na Listagem 20-16.

**Arquivo: src/lib.rs**

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

<a id="listagem-20-16"></a>

[Listagem 20-16](#listagem-20-16): Implementando a trait `Add` em `Millimeters` para somar `Millimeters` e `Meters`

Para somar `Millimeters` e `Meters`, especificamos `impl Add<Meters>` para definir o valor do parâmetro de tipo `Rhs` em vez de usar o padrão `Self`.

Você usará parâmetros de tipo padrão principalmente de duas formas:

1. Estender um tipo sem quebrar código existente
2. Permitir personalização em casos específicos que a maioria dos usuários não precisará

A trait `Add` da biblioteca padrão exemplifica o segundo propósito: em geral você soma tipos iguais, mas `Add` permite ir além. Usar um parâmetro de tipo padrão na definição de `Add` significa que na maior parte do tempo não precisamos especificar o parâmetro extra. Ou seja, evita-se um pouco de boilerplate de implementação, facilitando o uso da trait.

O primeiro propósito é parecido com o segundo, mas ao contrário: se quiser adicionar um parâmetro de tipo a uma trait existente, pode dar um padrão para estender a funcionalidade sem quebrar implementações já existentes.

## Desambiguando métodos com o mesmo nome

Nada em Rust impede que uma trait tenha um método com o mesmo nome de outra trait, nem impede implementar ambas as traits no mesmo tipo. Também é possível implementar um método diretamente no tipo com o mesmo nome dos métodos das traits.

Ao chamar métodos com o mesmo nome, você precisa dizer a Rust qual usar. Considere o código da Listagem 20-17, em que definimos duas traits, `Pilot` e `Wizard`, ambas com um método `fly`. Depois implementamos as duas traits no tipo `Human`, que já tem um método `fly` implementado diretamente. Cada método `fly` faz algo diferente.

**Arquivo: src/main.rs**

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

<a id="listagem-20-17"></a>

[Listagem 20-17](#listagem-20-17): Duas traits com método `fly` implementadas no tipo `Human`, e um método `fly` implementado diretamente em `Human`

Quando chamamos `fly` em uma instância de `Human`, o compilador usa por padrão o método implementado diretamente no tipo, como na Listagem 20-18.

**Arquivo: src/main.rs**

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    person.fly();
}
```

<a id="listagem-20-18"></a>

[Listagem 20-18](#listagem-20-18): Chamando `fly` em uma instância de `Human`

Executar este código imprime `*waving arms furiously*`, mostrando que Rust chamou o método `fly` implementado diretamente em `Human`.

Para chamar os métodos `fly` da trait `Pilot` ou da trait `Wizard`, precisamos de sintaxe mais explícita para indicar qual `fly` queremos. A Listagem 20-19 demonstra essa sintaxe.

**Arquivo: src/main.rs**

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

<a id="listagem-20-19"></a>

[Listagem 20-19](#listagem-20-19): Especificando qual método `fly` de trait queremos chamar

Especificar o nome da trait antes do nome do método deixa claro para Rust qual implementação de `fly` queremos. Também poderíamos escrever `Human::fly(&person)`, equivalente ao `person.fly()` da Listagem 20-19, mas é um pouco mais longo quando não precisamos desambiguar.

Executar este código imprime o seguinte:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.46s
     Running `target/debug/traits-example`
This is your captain speaking.
Up!
*waving arms furiously*
```

Como o método `fly` recebe um parâmetro `self`, se tivéssemos dois _tipos_ que implementam uma _trait_, Rust poderia descobrir qual implementação usar com base no tipo de `self`.

Porém, funções associadas que não são métodos não têm parâmetro `self`. Quando vários tipos ou traits definem funções não-método com o mesmo nome, Rust nem sempre sabe qual tipo você quer, a menos que use sintaxe totalmente qualificada. Por exemplo, na Listagem 20-20 criamos uma trait para um abrigo de animais que quer nomear todos os filhotes de cachorro de Spot. Fazemos uma trait `Animal` com uma função associada não-método `baby_name`. A trait `Animal` é implementada para a struct `Dog`, na qual também fornecemos diretamente uma função associada não-método `baby_name`.

**Arquivo: src/main.rs**

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

<a id="listagem-20-20"></a>

[Listagem 20-20](#listagem-20-20): Trait com função associada e tipo com função associada de mesmo nome que também implementa a trait

Implementamos em `baby_name` associada a `Dog` a lógica de nomear todos os filhotes de Spot. O tipo `Dog` também implementa a trait `Animal`, que descreve características de todos os animais. Filhotes de cachorro são chamados de puppies, expresso na implementação de `Animal` para `Dog` na função `baby_name` associada à trait `Animal`.

Em `main`, chamamos `Dog::baby_name`, que chama a função associada definida diretamente em `Dog`. Este código imprime:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.54s
     Running `target/debug/traits-example`
A baby dog is called a Spot
```

Essa saída não é o que queríamos. Queremos chamar a função `baby_name` da trait `Animal` implementada em `Dog`, para imprimir `A baby dog is called a puppy`. A técnica de especificar o nome da trait que usamos na Listagem 20-19 não ajuda aqui; se mudarmos `main` para o código da Listagem 20-21, teremos erro de compilação.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```

<a id="listagem-20-21"></a>

[Listagem 20-21](#listagem-20-21): Tentativa de chamar `baby_name` da trait `Animal`, mas Rust não sabe qual implementação usar

Como `Animal::baby_name` não tem parâmetro `self`, e pode haver outros tipos que implementam `Animal`, Rust não descobre qual implementação de `Animal::baby_name` queremos. Receberemos este erro do compilador:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding `impl` type
  --> src/main.rs:20:43
   |
 2 |     fn baby_name() -> String;
   |     ------------------------- `Animal::baby_name` defined here
...
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
   |                                           +++++++       +

For more information about this error, try `rustc --explain E0790`.
error: could not compile `traits-example` (bin "traits-example") due to 1 previous error
```

Para desambiguar e dizer a Rust que queremos a implementação de `Animal` para `Dog` — e não para outro tipo — usamos sintaxe totalmente qualificada. A Listagem 20-22 mostra como.

**Arquivo: src/main.rs**

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

<a id="listagem-20-22"></a>

[Listagem 20-22](#listagem-20-22): Usando sintaxe totalmente qualificada para chamar `baby_name` da trait `Animal` como implementada em `Dog`

Fornecemos a Rust uma anotação de tipo entre colchetes angulares, indicando que queremos chamar `baby_name` da trait `Animal` como implementada em `Dog`, tratando o tipo `Dog` como `Animal` nesta chamada. Este código imprime o que queremos:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/traits-example`
A baby dog is called a puppy
```

Em geral, a sintaxe totalmente qualificada é definida assim:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Para funções associadas que não são métodos, não haveria `receiver`: só a lista dos demais argumentos. Você poderia usar sintaxe totalmente qualificada em todo lugar em que chama funções ou métodos. Porém, pode omitir qualquer parte que Rust consiga inferir de outras informações do programa. Só precisa dessa sintaxe mais verbosa quando há várias implementações com o mesmo nome e Rust precisa de ajuda para identificar qual chamar.

## Usando supertraits

Às vezes você escreve uma definição de trait que depende de outra: para um tipo implementar a primeira trait, quer exigir que ele também implemente a segunda. Assim, sua definição de trait pode usar os itens associados da segunda trait. A trait da qual sua definição depende chama-se _supertrait_ da sua trait.

Por exemplo, suponha que queremos uma trait `OutlinePrint` com um método `outline_print` que imprime um valor formatado emoldurado por asteriscos. Ou seja, dada uma struct `Point` que implementa a trait `Display` da biblioteca padrão para resultar em `(x, y)`, ao chamar `outline_print` em uma instância de `Point` com `1` em `x` e `3` em `y`, deve imprimir:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Na implementação de `outline_print`, queremos usar a funcionalidade de `Display`. Portanto, precisamos especificar que `OutlinePrint` só funciona para tipos que também implementam `Display` e fornecem o que `OutlinePrint` precisa. Fazemos isso na definição da trait com `OutlinePrint: Display`. Essa técnica é parecida com adicionar um trait bound à trait. A Listagem 20-23 mostra uma implementação de `OutlinePrint`.

**Arquivo: src/main.rs**

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {output} *");
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

<a id="listagem-20-23"></a>

[Listagem 20-23](#listagem-20-23): Implementando a trait `OutlinePrint`, que exige funcionalidade de `Display`

Como especificamos que `OutlinePrint` exige a trait `Display`, podemos usar `to_string`, implementado automaticamente para qualquer tipo que implementa `Display`. Se tentássemos usar `to_string` sem dois pontos e a trait `Display` após o nome da trait, teríamos erro dizendo que nenhum método `to_string` foi encontrado para o tipo `&Self` no escopo atual.

Vejamos o que acontece ao tentar implementar `OutlinePrint` em um tipo que não implementa `Display`, como a struct `Point`:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {output} *");
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 1, y: 3 };
    p.outline_print();
}
```

Recebemos um erro dizendo que `Display` é exigida mas não implementada:

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0277]: `Point` doesn't implement `std::fmt::Display`
  --> src/main.rs:20:23
   |
20 | impl OutlinePrint for Point {}
   |                       ^^^^^ the trait `std::fmt::Display` is not implemented for `Point`
   |
note: required by a bound in `OutlinePrint`
  --> src/main.rs:3:21
   |
 3 | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in `OutlinePrint`

error[E0277]: `Point` doesn't implement `std::fmt::Display`
  --> src/main.rs:24:7
   |
24 |     p.outline_print();
   |       ^^^^^^^^^^^^^ the trait `std::fmt::Display` is not implemented for `Point`
   |
note: required by a bound in `OutlinePrint::outline_print`
  --> src/main.rs:3:21
   |
 3 | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in `OutlinePrint::outline_print`
 4 |     fn outline_print(&self) {
   |        ------------- required by a bound in this associated function

For more information about this error, try `rustc --explain E0277`.
error: could not compile `traits-example` (bin "traits-example") due to 2 previous errors
```

Para corrigir, implementamos `Display` em `Point` e satisfazemos a restrição que `OutlinePrint` exige:

**Arquivo: src/main.rs**

```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

Depois, implementar `OutlinePrint` em `Point` compila com sucesso, e podemos chamar `outline_print` em uma instância de `Point` para exibi-la dentro de um contorno de asteriscos.

## Implementando traits externas com o padrão newtype

Na seção Implementando uma trait em um tipo do Capítulo 10, mencionamos a regra do órfão: só podemos implementar uma trait em um tipo se a trait ou o tipo — ou ambos — forem locais ao nosso crate. É possível contornar essa restrição com o padrão newtype, criando um novo tipo em uma tuple struct. (Vimos tuple structs em Criando tipos diferentes com tuple structs do Capítulo 5.) A tuple struct terá um campo e será um envoltório fino do tipo para o qual queremos implementar a trait. O tipo envoltório é local ao nosso crate, e podemos implementar a trait nele. _Newtype_ é um termo originário da linguagem Haskell. Não há penalidade de desempenho em runtime por usar este padrão, e o tipo envoltório desaparece em tempo de compilação.

Como exemplo, suponha que queremos implementar `Display` em `Vec<T>`, o que a regra do órfão impede diretamente porque a trait `Display` e o tipo `Vec<T>` são definidos fora do nosso crate. Podemos fazer uma struct `Wrapper` que guarda uma instância de `Vec<T>`; depois implementamos `Display` em `Wrapper` e usamos o valor `Vec<T>`, como na Listagem 20-24.

**Arquivo: src/main.rs**

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {w}");
}
```

<a id="listagem-20-24"></a>

[Listagem 20-24](#listagem-20-24): Criando um tipo `Wrapper` em torno de `Vec<String>` para implementar `Display`

A implementação de `Display` usa `self.0` para acessar o `Vec<T>` interno, porque `Wrapper` é uma tuple struct e `Vec<T>` é o item no índice 0 da tupla. Depois podemos usar a funcionalidade de `Display` em `Wrapper`.

A desvantagem é que `Wrapper` é um tipo novo e não tem os métodos do valor que guarda. Teríamos de implementar todos os métodos de `Vec<T>` diretamente em `Wrapper`, delegando a `self.0`, para tratar `Wrapper` exatamente como `Vec<T>`. Se quisermos que o novo tipo tenha todos os métodos do tipo interno, implementar a trait `Deref` em `Wrapper` para retornar o tipo interno seria uma solução (discutimos `Deref` em Tratando smart pointers como referências comuns com a trait `Deref` do Capítulo 15). Se não quisermos que `Wrapper` tenha todos os métodos do tipo interno — por exemplo, para restringir o comportamento de `Wrapper` — implementamos manualmente só os métodos desejados.

Este padrão newtype também é útil mesmo sem traits envolvidas. Mudemos o foco e vejamos formas avançadas de interagir com o sistema de tipos de Rust.
