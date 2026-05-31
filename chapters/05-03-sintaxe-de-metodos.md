---
title: "Métodos"
chapter_code: 05-03
slug: sintaxe-de-metodos
---

# Métodos

Métodos são semelhantes a funções: declaramos com a palavra-chave `fn` e um nome, podem ter parâmetros e um valor de retorno, e contêm algum código que é executado quando o método é chamado de outro lugar. Diferentemente das funções, métodos são definidos no contexto de uma struct (ou de um enum ou de um trait object, que cobriremos no [Capítulo 6](/livro/cap06-00-enums-e-pattern-matching) e no [Capítulo 18](/livro/cap18-00-recursos-de-programacao-orientada-a-objetos), respectivamente), e seu primeiro parâmetro é sempre `self`, que representa a instância da struct na qual o método está sendo chamado.

### Sintaxe de métodos

Vamos alterar a função `area` que recebe uma instância de `Rectangle` como parâmetro e, em vez disso, criar um método `area` definido na struct `Rectangle`, como mostrado na Listagem 5-13.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

<a id="listagem-5-13"></a>

[Listagem 5-13](#listagem-5-13): Definindo um método `area` na struct `Rectangle`

Para definir a função no contexto de `Rectangle`, iniciamos um bloco `impl` (_implementation_) para `Rectangle`. Tudo dentro deste bloco `impl` estará associado ao tipo `Rectangle`. Em seguida, movemos a função `area` para dentro das chaves do `impl` e alteramos o primeiro (e, neste caso, único) parâmetro para `self` na assinatura e em todo o corpo. Em `main`, onde chamávamos a função `area` e passávamos `rect1` como argumento, podemos, em vez disso, usar a _sintaxe de método_ para chamar o método `area` na nossa instância de `Rectangle`. A sintaxe de método vem depois de uma instância: adicionamos um ponto seguido do nome do método, parênteses e quaisquer argumentos.

Na assinatura de `area`, usamos `&self` em vez de `rectangle: &Rectangle`. O `&self` é, na verdade, uma abreviação de `self: &Self`. Dentro de um bloco `impl`, o tipo `Self` é um alias para o tipo para o qual o bloco `impl` foi escrito. Métodos precisam ter um parâmetro chamado `self` do tipo `Self` como primeiro parâmetro, então o Rust permite abreviar isso usando apenas o nome `self` na primeira posição de parâmetro. Observe que ainda precisamos usar `&` na frente do atalho `self` para indicar que este método faz borrow da instância `Self`, assim como fizemos em `rectangle: &Rectangle`. Métodos podem tomar ownership de `self`, fazer borrow imutável de `self`, como fizemos aqui, ou fazer borrow mutável de `self`, assim como podem fazer com qualquer outro parâmetro.

Escolhemos `&self` aqui pelo mesmo motivo de termos usado `&Rectangle` na versão com função: não queremos tomar ownership e queremos apenas ler os dados na struct, não escrever neles. Se quiséssemos alterar a instância na qual chamamos o método como parte do que o método faz, usaríamos `&mut self` como primeiro parâmetro. Ter um método que toma ownership da instância usando apenas `self` como primeiro parâmetro é raro; essa técnica costuma ser usada quando o método transforma `self` em outra coisa e você quer impedir que o chamador use a instância original depois da transformação.

O principal motivo para usar métodos em vez de funções, além de fornecer sintaxe de método e não precisar repetir o tipo de `self` na assinatura de cada método, é a organização. Colocamos tudo o que podemos fazer com uma instância de um tipo em um bloco `impl`, em vez de fazer futuros usuários do nosso código procurarem as capacidades de `Rectangle` em vários lugares da biblioteca que fornecemos.

Observe que podemos escolher dar a um método o mesmo nome de um dos campos da struct. Por exemplo, podemos definir um método em `Rectangle` que também se chama `width`:

**Arquivo: src/main.rs**

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```

Aqui, escolhemos fazer o método `width` retornar `true` se o valor no campo `width` da instância for maior que `0` e `false` se o valor for `0`: podemos usar um campo dentro de um método de mesmo nome para qualquer finalidade. Em `main`, quando escrevemos `rect1.width()` com parênteses, o Rust sabe que queremos o método `width`. Quando não usamos parênteses, o Rust sabe que queremos o campo `width`.

Muitas vezes, mas nem sempre, quando damos a um método o mesmo nome de um campo, queremos que ele apenas retorne o valor do campo e não faça mais nada. Métodos assim são chamados de _getters_, e o Rust não os implementa automaticamente para campos de struct como algumas outras linguagens fazem. Getters são úteis porque você pode tornar o campo privado, mas o método público, e assim permitir acesso somente leitura a esse campo como parte da API pública do tipo. Discutiremos o que são público e privado e como designar um campo ou método como público ou privado no Capítulo 7.

> ### Onde está o operador `->`?
>
> Em C e C++, dois operadores diferentes são usados para chamar métodos: você usa `.` se estiver chamando um método diretamente no objeto e `->` se estiver chamando o método em um ponteiro para o objeto e precisar desreferenciar o ponteiro primeiro. Em outras palavras, se `object` for um ponteiro, `object->something()` é semelhante a `(*object).something()`.
>
> O Rust não tem um equivalente ao operador `->`; em vez disso, o Rust tem um recurso chamado _referenciamento e desreferenciamento automáticos_. Chamar métodos é um dos poucos lugares em Rust com esse comportamento.
>
> Veja como funciona: quando você chama um método com `object.something()`, o Rust adiciona automaticamente `&`, `&mut` ou `*` para que `object` corresponda à assinatura do método. Em outras palavras, o seguinte é o mesmo:
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> A primeira forma parece muito mais limpa. Esse comportamento de referenciamento automático funciona porque métodos têm um receptor claro — o tipo de `self`. Dado o receptor e o nome de um método, o Rust consegue determinar com certeza se o método está lendo (`&self`), mutando (`&mut self`) ou consumindo (`self`). O fato de o Rust tornar o borrowing implícito para receptores de métodos é uma parte importante de tornar o ownership ergonômico na prática.

### Métodos com mais parâmetros

Vamos praticar o uso de métodos implementando um segundo método na struct `Rectangle`. Desta vez, queremos que uma instância de `Rectangle` receba outra instância de `Rectangle` e retorne `true` se a segunda `Rectangle` couber completamente dentro de `self` (a primeira `Rectangle`); caso contrário, deve retornar `false`. Ou seja, depois de definirmos o método `can_hold`, queremos poder escrever o programa da Listagem 5-14.

**Arquivo: src/main.rs**

```rust
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<a id="listagem-5-14"></a>

[Listagem 5-14](#listagem-5-14): Usando o método `can_hold`, ainda não escrito

A saída esperada seria parecida com o seguinte, porque ambas as dimensões de `rect2` são menores que as dimensões de `rect1`, mas `rect3` é mais largo que `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir um método, então ele ficará dentro do bloco `impl Rectangle`. O nome do método será `can_hold`, e ele receberá um borrow imutável de outra `Rectangle` como parâmetro. Podemos dizer qual será o tipo do parâmetro olhando o código que chama o método: `rect1.can_hold(&rect2)` passa `&rect2`, que é um borrow imutável de `rect2`, uma instância de `Rectangle`. Isso faz sentido porque só precisamos ler `rect2` (em vez de escrever, o que exigiria um borrow mutável), e queremos que `main` mantenha o ownership de `rect2` para que possamos usá-la novamente depois de chamar o método `can_hold`. O valor de retorno de `can_hold` será um booleano, e a implementação verificará se a largura e a altura de `self` são maiores que a largura e a altura da outra `Rectangle`, respectivamente. Vamos adicionar o novo método `can_hold` ao bloco `impl` da Listagem 5-13, mostrado na Listagem 5-15.

**Arquivo: src/main.rs**

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<a id="listagem-5-15"></a>

[Listagem 5-15](#listagem-5-15): Implementando o método `can_hold` em `Rectangle`, que recebe outra instância de `Rectangle` como parâmetro

Quando executamos este código com a função `main` da Listagem 5-14, obteremos a saída desejada. Métodos podem receber vários parâmetros que adicionamos à assinatura depois do parâmetro `self`, e esses parâmetros funcionam como parâmetros em funções.

### Funções associadas

Todas as funções definidas dentro de um bloco `impl` são chamadas de _funções associadas_ porque estão associadas ao tipo nomeado depois do `impl`. Podemos definir funções associadas que não têm `self` como primeiro parâmetro (e, portanto, não são métodos) porque não precisam de uma instância do tipo para funcionar. Já usamos uma função assim: a função `String::from`, definida no tipo `String`.

Funções associadas que não são métodos costumam ser usadas como construtores que retornarão uma nova instância da struct. Muitas vezes são chamadas de `new`, mas `new` não é um nome especial e não está embutido na linguagem. Por exemplo, poderíamos fornecer uma função associada chamada `square` que teria um parâmetro de dimensão e usaria esse valor tanto para largura quanto para altura, facilitando a criação de um `Rectangle` quadrado em vez de ter que especificar o mesmo valor duas vezes:

**Arquivo: src/main.rs**

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

As palavras-chave `Self` no tipo de retorno e no corpo da função são aliases para o tipo que aparece depois da palavra-chave `impl`, que neste caso é `Rectangle`.

Para chamar essa função associada, usamos a sintaxe `::` com o nome da struct; `let sq = Rectangle::square(3);` é um exemplo. Essa função está no namespace da struct: a sintaxe `::` é usada tanto para funções associadas quanto para namespaces criados por módulos. Discutiremos módulos no Capítulo 7.

### Vários blocos `impl`

Cada struct pode ter vários blocos `impl`. Por exemplo, a Listagem 5-15 é equivalente ao código mostrado na Listagem 5-16, que tem cada método em seu próprio bloco `impl`.

**Arquivo: src/main.rs**

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<a id="listagem-5-16"></a>

[Listagem 5-16](#listagem-5-16): Reescrevendo a Listagem 5-15 usando vários blocos `impl`

Não há motivo para separar esses métodos em vários blocos `impl` aqui, mas essa sintaxe é válida. Veremos um caso em que vários blocos `impl` são úteis no Capítulo 10, quando discutirmos tipos genéricos e traits.

## Resumo

Structs permitem criar tipos personalizados que são significativos para o seu domínio. Usando structs, você pode manter pedaços de dados associados conectados entre si e nomear cada pedaço para deixar seu código claro. Em blocos `impl`, você pode definir funções associadas ao seu tipo, e métodos são um tipo de função associada que permite especificar o comportamento que instâncias das suas structs terão.

Mas structs não são a única forma de criar tipos personalizados: vamos passar para o recurso de enums do Rust para adicionar mais uma ferramenta à sua caixa de ferramentas.
