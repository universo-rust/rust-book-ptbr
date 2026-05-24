---
title: "Usando trait objects para abstrair comportamento compartilhado"
chapter_code: 18-02
slug: usando-trait-objects-para-abstrair-comportamento-compartilhado
---

# Usando Trait Objects para Abstrair Comportamento Compartilhado

No Capítulo 8, vetores só armazenam um tipo. Na Listagem 8-9 usamos o enum `SpreadsheetCell` com variantes para inteiros, floats e texto — tipos diferentes na mesma linha. Funciona quando o conjunto de tipos é fixo em tempo de compilação.

Às vezes queremos que usuários da biblioteca estendam os tipos válidos. Criaremos um exemplo de interface gráfica (GUI) que percorre itens e chama `draw` em cada um. Crate `gui` com `Button`, `TextField` etc.; usuários criam tipos próprios, como `Image` ou `SelectBox`.

Ao escrever a biblioteca, não sabemos todos os tipos futuros, mas sabemos que `gui` precisa rastrear valores de tipos diferentes e chamar `draw` — sem saber exatamente o que `draw` faz, só que existe.

Em linguagem com herança, poderíamos ter classe `Component` com `draw`; `Button`, `Image`, `SelectBox` herdariam e poderiam sobrescrever. O framework trataria tudo como `Component`. Sem herança em Rust, precisamos de outra estrutura.

## Definindo uma trait para comportamento comum

Definimos a trait `Draw` com método `draw` e um vetor de trait objects. Um _trait object_ aponta para uma instância que implementa a trait e uma tabela para buscar métodos em tempo de execução. Criamos com ponteiro (`&` ou `Box<T>`), `dyn` e a trait (ver Tipos de tamanho dinâmico e a trait `Sized` no Capítulo 20). Trait objects substituem tipo genérico ou concreto; o compilador garante que valores nesse contexto implementam a trait.

Em Rust evitamos chamar structs e enums de “objetos”. Em outras linguagens, dados e comportamento costumam ser um só “objeto”. Trait objects não permitem adicionar dados à trait; servem para abstrair comportamento comum.

**Arquivo: src/lib.rs**

```rust
pub trait Draw {
    fn draw(&self);
}
```

[Listagem 18-3](#listagem-18-3): Definição da trait `Draw`

A Listagem 18-4 define `Screen` com `components: Vec<Box<dyn Draw>>` — trait object; qualquer tipo em `Box` que implemente `Draw`.

**Arquivo: src/lib.rs**

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

[Listagem 18-4](#listagem-18-4): `Screen` com vetor de trait objects que implementam `Draw`

Na Listagem 18-5, `run` chama `draw` em cada componente.

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

[Listagem 18-5](#listagem-18-5): Método `run` em `Screen` que chama `draw` em cada componente

Isso difere de struct com parâmetro de tipo genérico e trait bounds. Genérico substitui por um tipo concreto por vez; trait objects permitem vários tipos concretos em tempo de execução. A Listagem 18-6 mostra a alternativa com genéricos.

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

[Listagem 18-6](#listagem-18-6): Implementação alternativa de `Screen` com genéricos e trait bounds

Isso limita `Screen` a componentes todos `Button` ou todos `TextField`. Coleções homogêneas preferem genéricos (monomorfização em compile time).

Com trait objects, uma `Screen` pode ter `Box<Button>` e `Box<TextField>` no mesmo `Vec`. Vejamos a implementação e o desempenho.

## Implementando a trait

Adicionamos tipos que implementam `Draw`, como `Button` (Listagem 18-7).

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

[Listagem 18-7](#listagem-18-7): Struct `Button` que implementa `Draw`

`TextField` poderia ter `placeholder` além de largura e altura. Cada tipo implementa `draw` à sua maneira. `Button` pode ter outro `impl` com métodos de clique que não se aplicam a `TextField`.

Se alguém implementar `SelectBox` com `width`, `height` e `options`, também implementa `Draw` (Listagem 18-8).

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

[Listagem 18-8](#listagem-18-8): Outro crate usando `gui` e implementando `Draw` em `SelectBox`

O usuário cria `Screen`, adiciona `SelectBox` e `Button` em `Box` como trait objects e chama `run` (Listagem 18-9).

**Arquivo: src/main.rs**

```rust
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

[Listagem 18-9](#listagem-18-9): Trait objects para armazenar tipos diferentes que implementam a mesma trait

Ao escrever a biblioteca não sabíamos de `SelectBox`, mas `Screen` opera nele porque implementa `Draw`.

Preocupar-se com as mensagens a que um valor responde, não com o tipo concreto, lembra _duck typing_: se anda e grasna como pato, é pato. Em `run` da Listagem 18-5 não importa se é `Button` ou `SelectBox` — só chama `draw`. Com `Box<dyn Draw>`, exigimos valores com `draw`.

A vantagem sobre duck typing dinâmico: não verificamos métodos em runtime nem chamamos método inexistente. Rust não compila se os valores não implementam as traits exigidas.

A Listagem 18-10 tenta `String` como componente.

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

[Listagem 18-10](#listagem-18-10): Tentativa de usar tipo que não implementa a trait do trait object

Erro:

```bash
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

Ou passamos o tipo errado, ou implementamos `Draw` em `String`.

## Despacho dinâmico

Em Desempenho de código que usa generics no Capítulo 10, a monomorfização gera implementações não genéricas por tipo concreto — _despacho estático_: o compilador sabe qual método chamar em compile time. _Despacho dinâmico_: não sabe em compile time; em runtime descobre qual método chamar.

Com trait objects, Rust usa despacho dinâmico. Não conhece todos os tipos possíveis; em runtime usa ponteiros no trait object para chamar o método. Há custo em runtime que despacho estático não tem; também impede inlining e algumas otimizações. Há regras de _compatibilidade com `dyn`_ ([referência](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility)). Ganhamos flexibilidade nas Listagens 18-5 e 18-9 — trade-off a considerar.
