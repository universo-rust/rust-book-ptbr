---
title: "Tratando smart pointers como referências comuns com a trait `Deref`"
chapter_code: 15-02
slug: tratando-smart-pointers-como-referencias-com-a-trait-deref
---

# Tratando smart pointers como referências comuns com a trait `Deref`

Implementar a trait `Deref` permite personalizar o comportamento do _operador de dereferência_ `*` (não confundir com o operador de multiplicação ou glob). Ao implementar `Deref` de forma que um smart pointer possa ser tratado como uma referência comum, você pode escrever código que opera em referências e usar esse código com smart pointers também.

Primeiro veremos como o operador de dereferência funciona com referências comuns. Depois, tentaremos definir um tipo personalizado que se comporta como `Box<T>` e veremos por que o operador de dereferência não funciona como uma referência em nosso tipo recém-definido. Exploraremos como implementar a trait `Deref` torna possível que smart pointers funcionem de formas semelhantes a referências. Em seguida, olharemos o recurso de coerção de deref do Rust e como ele nos permite trabalhar com referências ou smart pointers.

### Seguindo a referência até o valor

Uma referência comum é um tipo de ponteiro, e uma forma de pensar em um ponteiro é como uma seta para um valor armazenado em outro lugar. Na Listagem 15-6, criamos uma referência a um valor `i32` e então usamos o operador de dereferência para seguir a referência até o valor.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<a id="listagem-15-6"></a>

[Listagem 15-6](#listagem-15-6): Usando o operador de dereferência para seguir uma referência a um valor `i32`

A variável `x` guarda um valor `i32` `5`. Definimos `y` igual a uma referência a `x`. Podemos afirmar que `x` é igual a `5`. No entanto, se quisermos fazer uma asserção sobre o valor em `y`, temos que usar `*y` para seguir a referência ao valor para o qual aponta (daí, _dereference_) para que o compilador possa comparar o valor real. Depois de dereferenciar `y`, temos acesso ao valor inteiro para o qual `y` aponta e podemos compará-lo com `5`.

Se tentássemos escrever `assert_eq!(5, y);` em vez disso, obteríamos este erro de compilação:

```console
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0277]: can't compare `{integer}` with `&{integer}`
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
  |
  = help: the trait `PartialEq<&{integer}>` is not implemented for `{integer}`
  = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
error: could not compile `deref-example` (bin "deref-example") due to 1 previous error
```

Comparar um número e uma referência a um número não é permitido porque são tipos diferentes. Devemos usar o operador de dereferência para seguir a referência ao valor para o qual aponta.

### Usando `Box<T>` como uma referência

Podemos reescrever o código da Listagem 15-6 para usar um `Box<T>` em vez de uma referência; o operador de dereferência usado no `Box<T>` na Listagem 15-7 funciona da mesma forma que o operador de dereferência usado na referência na Listagem 15-6.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<a id="listagem-15-7"></a>

[Listagem 15-7](#listagem-15-7): Usando o operador de dereferência em um `Box<i32>`

A principal diferença entre a Listagem 15-7 e a Listagem 15-6 é que aqui definimos `y` como uma instância de uma box apontando para uma cópia do valor de `x` em vez de uma referência apontando para o valor de `x`. Na última asserção, podemos usar o operador de dereferência para seguir o ponteiro da box da mesma forma que fizemos quando `y` era uma referência. Em seguida, exploraremos o que há de especial em `Box<T>` que nos permite usar o operador de dereferência definindo nosso próprio tipo de box.

### Definindo nosso próprio smart pointer

Vamos construir um tipo wrapper semelhante ao tipo `Box<T>` fornecido pela biblioteca padrão para experimentar como tipos smart pointer se comportam diferente de referências por padrão. Depois, veremos como adicionar a capacidade de usar o operador de dereferência.

> **Nota:** Há uma grande diferença entre o tipo `MyBox<T>` que estamos prestes a construir e o `Box<T>` real: nossa versão não armazenará seus dados na heap. Estamos focando este exemplo em `Deref`, então onde os dados estão realmente armazenados é menos importante que o comportamento semelhante a ponteiro.

O tipo `Box<T>` é definido como uma tuple struct com um elemento, então a Listagem 15-8 define um tipo `MyBox<T>` da mesma forma. Também definiremos uma função `new` para corresponder à função `new` definida em `Box<T>`.

**Arquivo: src/main.rs**

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {}
```

<a id="listagem-15-8"></a>

[Listagem 15-8](#listagem-15-8): Definindo um tipo `MyBox<T>`

Definimos uma struct chamada `MyBox` e declaramos um parâmetro genérico `T` porque queremos que nosso tipo guarde valores de qualquer tipo. O tipo `MyBox` é uma tuple struct com um elemento do tipo `T`. A função `MyBox::new` recebe um parâmetro do tipo `T` e retorna uma instância `MyBox` que guarda o valor passado.

Vamos tentar adicionar a função `main` da Listagem 15-7 à Listagem 15-8 e mudá-la para usar o tipo `MyBox<T>` que definimos em vez de `Box<T>`. O código na Listagem 15-9 não compilará, porque o Rust não sabe como dereferenciar `MyBox`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<a id="listagem-15-9"></a>

[Listagem 15-9](#listagem-15-9): Tentativa de usar `MyBox<T>` da mesma forma que usamos referências e `Box<T>`

Aqui está o erro de compilação resultante:

```console
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^ can't be dereferenced

For more information about this error, try `rustc --explain E0614`.
error: could not compile `deref-example` (bin "deref-example") due to 1 previous error
```

Nosso tipo `MyBox<T>` não pode ser dereferenciado porque não implementamos essa capacidade em nosso tipo. Para habilitar dereferenciação com o operador `*`, implementamos a trait `Deref`.

### Implementando a trait `Deref`

Como discutido em Implementando uma trait em um tipo no Capítulo 10, para implementar uma trait precisamos fornecer implementações para os métodos exigidos da trait. A trait `Deref`, fornecida pela biblioteca padrão, exige que implementemos um método chamado `deref` que empresta `self` e retorna uma referência aos dados internos. A Listagem 15-10 contém uma implementação de `Deref` para adicionar à definição de `MyBox<T>`.

**Arquivo: src/main.rs**

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<a id="listagem-15-10"></a>

[Listagem 15-10](#listagem-15-10): Implementando `Deref` em `MyBox<T>`

A sintaxe `type Target = T;` define um tipo associado para a trait `Deref` usar. Tipos associados são uma forma ligeiramente diferente de declarar um parâmetro genérico, mas você não precisa se preocupar com eles por agora; cobriremos em mais detalhes no Capítulo 20.

Preenchemos o corpo do método `deref` com `&self.0` para que `deref` retorne uma referência ao valor que queremos acessar com o operador `*`; lembre-se de Criando diferentes tipos com tuple structs no Capítulo 5 de que `.0` acessa o primeiro valor em uma tuple struct. A função `main` na Listagem 15-9 que chama `*` no valor `MyBox<T>` agora compila, e as asserções passam!

Sem a trait `Deref`, o compilador só pode dereferenciar referências `&`. O método `deref` dá ao compilador a capacidade de pegar um valor de qualquer tipo que implemente `Deref` e chamar o método `deref` para obter uma referência que ele sabe como dereferenciar.

Quando digitamos `*y` na Listagem 15-9, nos bastidores o Rust na verdade executou este código:

```rust
*(y.deref())
```

O Rust substitui o operador `*` por uma chamada ao método `deref` e então uma dereferência simples para que não precisemos pensar se precisamos ou não chamar o método `deref`. Este recurso do Rust nos permite escrever código que funciona de forma idêntica, quer tenhamos uma referência comum ou um tipo que implemente `Deref`.

A razão pela qual o método `deref` retorna uma referência a um valor, e que a dereferência simples fora dos parênteses em `*(y.deref())` ainda é necessária, tem a ver com o sistema de ownership. Se o método `deref` retornasse o valor diretamente em vez de uma referência ao valor, o valor seria movido para fora de `self`. Não queremos tomar posse do valor interno dentro de `MyBox<T>` neste caso nem na maioria dos casos em que usamos o operador de dereferência.

Note que o operador `*` é substituído por uma chamada ao método `deref` e então uma chamada ao operador `*` apenas uma vez, cada vez que usamos um `*` em nosso código. Como a substituição do operador `*` não recursa infinitamente, acabamos com dados do tipo `i32`, que corresponde ao `5` em `assert_eq!` na Listagem 15-9.

### Usando coerção de deref em funções e métodos

_Coerção de deref_ converte uma referência a um tipo que implementa a trait `Deref` em uma referência a outro tipo. Por exemplo, coerção de deref pode converter `&String` em `&str` porque `String` implementa a trait `Deref` de forma que retorna `&str`. Coerção de deref é uma conveniência que o Rust realiza em argumentos para funções e métodos, e funciona apenas em tipos que implementam a trait `Deref`. Acontece automaticamente quando passamos uma referência ao valor de um tipo particular como argumento para uma função ou método cujo parâmetro não corresponde ao tipo na definição da função ou método. Uma sequência de chamadas ao método `deref` converte o tipo que fornecemos no tipo que o parâmetro precisa.

A coerção de deref foi adicionada ao Rust para que programadores escrevendo chamadas de função e método não precisassem adicionar tantas referências e dereferências explícitas com `&` e `*`. O recurso de coerção de deref também nos permite escrever mais código que pode funcionar tanto para referências quanto para smart pointers.

Para ver a coerção de deref em ação, usaremos o tipo `MyBox<T>` que definimos na Listagem 15-8, bem como a implementação de `Deref` que adicionamos na Listagem 15-10. A Listagem 15-11 mostra a definição de uma função que tem um parâmetro string slice.

**Arquivo: src/main.rs**

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {}
```

<a id="listagem-15-11"></a>

[Listagem 15-11](#listagem-15-11): Uma função `hello` que tem o parâmetro `name` do tipo `&str`

Podemos chamar a função `hello` com um string slice como argumento, como `hello("Rust");`, por exemplo. A coerção de deref torna possível chamar `hello` com uma referência a um valor do tipo `MyBox<String>`, como mostrado na Listagem 15-12.

**Arquivo: src/main.rs**

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

<a id="listagem-15-12"></a>

[Listagem 15-12](#listagem-15-12): Chamando `hello` com uma referência a um valor `MyBox<String>`, o que funciona por causa da coerção de deref

Aqui estamos chamando a função `hello` com o argumento `&m`, que é uma referência a um valor `MyBox<String>`. Como implementamos a trait `Deref` em `MyBox<T>` na Listagem 15-10, o Rust pode transformar `&MyBox<String>` em `&String` chamando `deref`. A biblioteca padrão fornece uma implementação de `Deref` em `String` que retorna um string slice, e isso está na documentação da API de `Deref`. O Rust chama `deref` novamente para transformar o `&String` em `&str`, que corresponde à definição da função `hello`.

Se o Rust não implementasse coerção de deref, teríamos que escrever o código da Listagem 15-13 em vez do código da Listagem 15-12 para chamar `hello` com um valor do tipo `&MyBox<String>`.

**Arquivo: src/main.rs**

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

<a id="listagem-15-13"></a>

[Listagem 15-13](#listagem-15-13): O código que teríamos que escrever se o Rust não tivesse coerção de deref

O `(*m)` dereferencia o `MyBox<String>` em um `String`. Então, o `&` e `[..]` tomam um string slice do `String` que é igual à string inteira para corresponder à assinatura de `hello`. Este código sem coerção de deref é mais difícil de ler, escrever e entender com todos esses símbolos envolvidos. A coerção de deref permite que o Rust trate essas conversões para nós automaticamente.

Quando a trait `Deref` é definida para os tipos envolvidos, o Rust analisará os tipos e usará `Deref::deref` quantas vezes forem necessárias para obter uma referência que corresponda ao tipo do parâmetro. O número de vezes que `Deref::deref` precisa ser inserido é resolvido em tempo de compilação, então não há penalidade em tempo de execução por aproveitar a coerção de deref!

### Lidando com coerção de deref e referências mutáveis

De forma semelhante a como você usa a trait `Deref` para substituir o operador `*` em referências imutáveis, pode usar a trait `DerefMut` para substituir o operador `*` em referências mutáveis.

O Rust faz coerção de deref quando encontra tipos e implementações de trait nestes três casos:

1. De `&T` para `&U` quando `T: Deref<Target=U>`
2. De `&mut T` para `&mut U` quando `T: DerefMut<Target=U>`
3. De `&mut T` para `&U` quando `T: Deref<Target=U>`

Os dois primeiros casos são iguais, exceto que o segundo implementa mutabilidade. O primeiro caso afirma que, se você tem um `&T`, e `T` implementa `Deref` para algum tipo `U`, pode obter um `&U` de forma transparente. O segundo caso afirma que a mesma coerção de deref acontece para referências mutáveis.

O terceiro caso é mais complicado: o Rust também coercerá uma referência mutável em uma imutável. Mas o inverso _não_ é possível: referências imutáveis nunca serão coercidas em referências mutáveis. Por causa das regras de borrowing, se você tem uma referência mutável, essa referência mutável deve ser a única referência àqueles dados (caso contrário, o programa não compilaria). Converter uma referência mutável em uma referência imutável nunca quebrará as regras de borrowing. Converter uma referência imutável em uma referência mutável exigiria que a referência imutável inicial fosse a única referência imutável àqueles dados, mas as regras de borrowing não garantem isso. Portanto, o Rust não pode assumir que converter uma referência imutável em uma referência mutável é possível.
