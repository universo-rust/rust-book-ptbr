---
title: "Usando trait objects para abstrair comportamento compartilhado"
chapter_code: 18-02
slug: usando-trait-objects-para-abstrair-comportamento-compartilhado
---

# Usando trait objects para abstrair comportamento compartilhado

No Capítulo 8, mencionamos que uma limitação dos vetores é que eles só podem armazenar elementos de um único tipo. Criamos uma alternativa na Listagem 8-9 definindo um enum `SpreadsheetCell` com variantes para guardar inteiros, números de ponto flutuante e texto. Isso nos permitiu armazenar tipos diferentes de dados em cada célula e ainda ter um vetor representando uma linha de células. Essa é uma solução perfeitamente adequada quando os itens intercambiáveis pertencem a um conjunto fixo de tipos conhecido no momento da compilação.

Às vezes, porém, queremos que quem usa nossa biblioteca possa estender o conjunto de tipos válidos em uma determinada situação. Para mostrar como podemos fazer isso, vamos criar um exemplo de ferramenta de interface gráfica (GUI) que percorre uma lista de itens e chama um método `draw` em cada um para desenhá-lo na tela, uma técnica comum em ferramentas de GUI. Vamos criar um crate de biblioteca chamado `gui`, que contém a estrutura de uma biblioteca de interface gráfica. Esse crate poderia incluir alguns tipos para as pessoas usarem, como `Button` ou `TextField`. Além disso, usuários de `gui` vão querer criar seus próprios tipos desenháveis: uma pessoa poderia adicionar `Image`, por exemplo, e outra poderia adicionar `SelectBox`.

No momento em que escrevemos a biblioteca, não conseguimos saber nem definir todos os tipos que outras pessoas talvez queiram criar. Mas sabemos que `gui` precisa acompanhar muitos valores de tipos diferentes e chamar um método `draw` em cada um desses valores. Não precisamos saber exatamente o que acontecerá quando chamarmos `draw`; precisamos apenas saber que o valor terá esse método disponível.

Em uma linguagem com herança, poderíamos definir uma classe chamada `Component` com um método `draw`. Outras classes, como `Button`, `Image` e `SelectBox`, herdariam de `Component` e, portanto, herdariam o método `draw`. Cada uma poderia sobrescrever `draw` para definir seu próprio comportamento, mas o framework poderia tratar todos esses tipos como instâncias de `Component` e chamar `draw` neles. Como Rust não tem herança, precisamos de outra forma de estruturar a biblioteca `gui` para permitir que usuários criem novos tipos compatíveis com ela.

## Definindo uma trait para comportamento comum

Para implementar o comportamento que queremos em `gui`, vamos definir uma trait chamada `Draw` com um método chamado `draw`. Então, podemos definir um vetor que recebe trait objects. Um _trait object_ aponta tanto para uma instância de um tipo que implementa a trait especificada quanto para uma tabela usada para procurar, em tempo de execução, os métodos dessa trait naquele tipo. Criamos um trait object especificando algum tipo de ponteiro, como uma referência ou um smart pointer `Box<T>`, depois a palavra-chave `dyn` e, por fim, a trait relevante. Falaremos sobre por que trait objects precisam usar um ponteiro em ["Tipos de tamanho dinâmico e a trait `Sized`"](/livro/cap20-03-tipos-avancados#tipos-de-tamanho-dinamico-e-a-trait-sized), no Capítulo 20. Podemos usar trait objects no lugar de um tipo genérico ou concreto. Em qualquer lugar em que usamos um trait object, o sistema de tipos de Rust garante, em tempo de compilação, que qualquer valor usado naquele contexto implementa a trait do trait object. Como consequência, não precisamos conhecer todos os tipos possíveis em tempo de compilação.

Já mencionamos que, em Rust, evitamos chamar structs e enums de "objetos" para distingui-los dos objetos de outras linguagens. Em uma struct ou enum, os dados nos campos e o comportamento em blocos `impl` ficam separados; em outras linguagens, dados e comportamento combinados em um único conceito costumam receber o nome de objeto. Trait objects diferem dos objetos de outras linguagens porque não podemos adicionar dados a um trait object. Eles não são tão gerais quanto objetos em outras linguagens: sua finalidade específica é permitir abstração sobre comportamento comum.

A Listagem 18-3 mostra como definir uma trait chamada `Draw` com um método chamado `draw`.

**Arquivo: src/lib.rs**

```rust
pub trait Draw {
    fn draw(&self);
}
```

<a id="listagem-18-3"></a>

[Listagem 18-3](#listagem-18-3): Definição da trait `Draw`

Essa sintaxe deve parecer familiar a partir das discussões sobre como definir traits no Capítulo 10. Agora vem uma sintaxe nova: a Listagem 18-4 define uma struct chamada `Screen`, que guarda um vetor chamado `components`. Esse vetor tem o tipo `Vec<Box<dyn Draw>>`, que é um trait object; ele representa qualquer tipo dentro de um `Box` que implemente a trait `Draw`.

**Arquivo: src/lib.rs**

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

<a id="listagem-18-4"></a>

[Listagem 18-4](#listagem-18-4): Struct `Screen` com um vetor de trait objects que implementam a trait `Draw`

Na struct `Screen`, definiremos um método chamado `run` que chama o método `draw` em cada item de `components`, como mostra a Listagem 18-5.

**Arquivo: src/lib.rs**

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<a id="listagem-18-5"></a>

[Listagem 18-5](#listagem-18-5): Método `run` em `Screen` que chama `draw` em cada componente

Isso funciona de modo diferente de definir uma struct que usa um parâmetro de tipo genérico com trait bounds. Um parâmetro de tipo genérico só pode ser substituído por um tipo concreto por vez, enquanto trait objects permitem que vários tipos concretos ocupem o lugar do trait object em tempo de execução. Por exemplo, poderíamos ter definido a struct `Screen` usando um tipo genérico e uma trait bound, como na Listagem 18-6.

**Arquivo: src/lib.rs**

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
where
    T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<a id="listagem-18-6"></a>

[Listagem 18-6](#listagem-18-6): Uma implementação alternativa de `Screen` usando generics e trait bounds

Essa versão nos restringe a uma instância de `Screen` que tenha uma lista de componentes todos do tipo `Button` ou todos do tipo `TextField`. Se você só precisa de coleções homogêneas, usar generics e trait bounds é preferível, porque as definições serão monomorfizadas em tempo de compilação para usar os tipos concretos.

Por outro lado, com o método usando trait objects, uma instância de `Screen` pode conter um `Vec<Box<dyn Draw>>` com um `Box<Button>` e também um `Box<TextField>`. Vamos ver como isso funciona e, depois, falar sobre as implicações de desempenho em tempo de execução.

## Implementando a trait

Agora vamos adicionar alguns tipos que implementam a trait `Draw`. Forneceremos o tipo `Button`. Mais uma vez, implementar uma biblioteca GUI de verdade está fora do escopo deste livro, então o método `draw` não terá uma implementação útil no corpo. Para imaginar como a implementação poderia ser, uma struct `Button` talvez tivesse campos para `width`, `height` e `label`, como mostra a Listagem 18-7.

**Arquivo: src/lib.rs**

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // código para desenhar um botão
    }
}
```

<a id="listagem-18-7"></a>

[Listagem 18-7](#listagem-18-7): Struct `Button` que implementa a trait `Draw`

Os campos `width`, `height` e `label` de `Button` serão diferentes dos campos de outros componentes. Por exemplo, um tipo `TextField` poderia ter esses mesmos campos, além de um campo `placeholder`. Cada tipo que quisermos desenhar na tela implementará a trait `Draw`, mas usará um código diferente no método `draw` para definir como aquele tipo específico deve ser desenhado, como `Button` faz aqui, sem o código real de GUI. O tipo `Button`, por exemplo, poderia ter outro bloco `impl` com métodos relacionados ao que acontece quando uma pessoa clica no botão. Esses métodos não se aplicariam a tipos como `TextField`.

Se alguém usando nossa biblioteca decidir implementar uma struct `SelectBox` com campos `width`, `height` e `options`, também implementaria a trait `Draw` para o tipo `SelectBox`, como mostra a Listagem 18-8.

**Arquivo: src/main.rs**

```rust
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // código para desenhar uma caixa de seleção
    }
}
```

<a id="listagem-18-8"></a>

[Listagem 18-8](#listagem-18-8): Outro crate usando `gui` e implementando a trait `Draw` em `SelectBox`

Agora, quem usa nossa biblioteca pode escrever a função `main` criando uma instância de `Screen`. Nessa instância, pode adicionar um `SelectBox` e um `Button`, colocando cada um em um `Box<T>` para transformá-los em trait objects. Depois, pode chamar o método `run` na instância de `Screen`, que chamará `draw` em cada componente. A Listagem 18-9 mostra essa implementação.

**Arquivo: src/main.rs**

```rust
use gui::Draw;
use gui::{Button, Screen};

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // código para desenhar uma caixa de seleção
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

<a id="listagem-18-9"></a>

[Listagem 18-9](#listagem-18-9): Usando trait objects para armazenar valores de tipos diferentes que implementam a mesma trait

Quando escrevemos a biblioteca, não sabíamos que alguém poderia adicionar o tipo `SelectBox`, mas nossa implementação de `Screen` conseguiu trabalhar com esse novo tipo e desenhá-lo porque `SelectBox` implementa a trait `Draw`, o que significa que implementa o método `draw`.

Esse conceito, de se preocupar apenas com as mensagens às quais um valor responde em vez de se preocupar com seu tipo concreto, é parecido com _duck typing_ em linguagens de tipagem dinâmica: se anda como pato e grasna como pato, então deve ser um pato! Na implementação de `run` em `Screen`, na Listagem 18-5, `run` não precisa saber qual é o tipo concreto de cada componente. Ele não verifica se um componente é uma instância de `Button` ou de `SelectBox`; simplesmente chama o método `draw` no componente. Ao especificar `Box<dyn Draw>` como o tipo dos valores no vetor `components`, definimos que `Screen` precisa de valores nos quais possamos chamar o método `draw`.

A vantagem de usar trait objects e o sistema de tipos de Rust para escrever código parecido com duck typing é que nunca precisamos verificar em tempo de execução se um valor implementa determinado método, nem nos preocupar em receber erros por chamar um método que o valor não implementa. Rust não compilará nosso código se os valores não implementarem as traits exigidas pelos trait objects.

Por exemplo, a Listagem 18-10 mostra o que acontece se tentarmos criar uma `Screen` usando uma `String` como componente.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![Box::new(String::from("Hi"))],
    };

    screen.run();
}
```

<a id="listagem-18-10"></a>

[Listagem 18-10](#listagem-18-10): Tentativa de usar um tipo que não implementa a trait do trait object

Receberemos este erro porque `String` não implementa a trait `Draw`:

```console
$ cargo run
   Compiling gui v0.1.0 (file:///projects/gui)
error[E0277]: the trait bound `String: Draw` is not satisfied
 --> src/main.rs:5:26
  |
5 |         components: vec![Box::new(String::from("Hi"))],
  |                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Draw` is not implemented for `String`
  |
  = help: the trait `Draw` is implemented for `Button`
  = note: required for the cast from `Box<String>` to `Box<dyn Draw>`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `gui` (bin "gui") due to 1 previous error
```

Esse erro nos informa que ou estamos passando para `Screen` algo que não pretendíamos passar, e portanto devemos passar outro tipo, ou precisamos implementar `Draw` para `String`, para que `Screen` possa chamar `draw` nela.

## Despacho dinâmico

Lembre-se da discussão sobre monomorfização em ["Desempenho do código que usa generics"](/livro/cap10-01-tipos-de-dados-genericos#desempenho-do-codigo-que-usa-generics), no Capítulo 10: o compilador gera implementações não genéricas de funções e métodos para cada tipo concreto que usamos no lugar de um parâmetro de tipo genérico. O código resultante da monomorfização usa _despacho estático_, que acontece quando o compilador sabe, em tempo de compilação, qual método você está chamando. Isso se opõe ao _despacho dinâmico_, que acontece quando o compilador não consegue saber em tempo de compilação qual método você está chamando. Em casos de despacho dinâmico, o compilador emite código que, em tempo de execução, descobre qual método chamar.

Quando usamos trait objects, Rust precisa usar despacho dinâmico. O compilador não conhece todos os tipos que podem ser usados com o código que usa trait objects, então não sabe qual método implementado em qual tipo deve chamar. Em vez disso, em tempo de execução, Rust usa os ponteiros dentro do trait object para saber qual método chamar. Essa busca tem um custo em tempo de execução que não existe com despacho estático. O despacho dinâmico também impede que o compilador escolha embutir (_inline_) o código de um método, o que por sua vez impede algumas otimizações. Além disso, Rust tem regras sobre onde você pode ou não usar despacho dinâmico, chamadas de _compatibilidade com `dyn`_. Essas regras estão fora do escopo desta discussão, mas você pode ler mais sobre elas [na referência](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility). Ainda assim, ganhamos flexibilidade extra no código que escrevemos na Listagem 18-5 e conseguimos dar suporte ao caso da Listagem 18-9, então esse é um tradeoff a considerar.
