---
title: "Definindo comportamento compartilhado com traits"
chapter_code: 10-02
slug: definindo-comportamento-compartilhado-com-traits
---

# Definindo comportamento compartilhado com traits

Uma _trait_ define a funcionalidade que um tipo particular tem e pode compartilhar com outros tipos. Podemos usar traits para definir comportamento compartilhado de forma abstrata. Podemos usar _trait bounds_ para especificar que um tipo genérico pode ser qualquer tipo que tenha determinado comportamento.

> Nota: Traits são semelhantes a um recurso frequentemente chamado de _interfaces_ em outras linguagens, embora com algumas diferenças.

### Definindo uma trait

O comportamento de um tipo consiste nos métodos que podemos chamar nesse tipo. Tipos diferentes compartilham o mesmo comportamento se pudermos chamar os mesmos métodos em todos eles. Definições de trait são uma forma de agrupar assinaturas de método para definir um conjunto de comportamentos necessários para algum propósito.

Por exemplo, digamos que temos várias structs que armazenam vários tipos e quantidades de texto: uma struct `NewsArticle` que armazena uma matéria em um local particular e uma `SocialPost` que pode ter, no máximo, 280 caracteres, junto com metadados que indicam se foi uma postagem nova, um repost ou uma resposta a outra postagem.

Queremos criar uma crate de biblioteca chamada `aggregator` que possa exibir resumos de dados que possam estar armazenados em uma instância de `NewsArticle` ou `SocialPost`. Para isso, precisamos de um resumo de cada tipo e solicitaremos esse resumo chamando um método `summarize` em uma instância. A [Listagem 10-12](#listagem-10-12) mostra a definição de uma trait pública `Summary` que expressa esse comportamento.

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

<a id="listagem-10-12"></a>

[Listagem 10-12](#listagem-10-12): Uma trait `Summary` que consiste no comportamento fornecido por um método `summarize`

Aqui, declaramos uma trait usando a palavra-chave `trait` e depois o nome da trait, que neste caso é `Summary`. Também declaramos a trait como `pub` para que crates que dependem desta crate possam usar esta trait também, como veremos em alguns exemplos. Dentro das chaves, declaramos as assinaturas de método que descrevem os comportamentos dos tipos que implementam esta trait, que neste caso é `fn summarize(&self) -> String`.

Depois da assinatura do método, em vez de fornecer uma implementação entre chaves, usamos um ponto e vírgula. Cada tipo que implementa esta trait deve fornecer seu próprio comportamento personalizado para o corpo do método. O compilador garantirá que qualquer tipo que tenha a trait `Summary` terá o método `summarize` definido com exatamente esta assinatura.

Uma trait pode ter vários métodos em seu corpo: as assinaturas de método são listadas uma por linha, e cada linha termina com ponto e vírgula.

### Implementando uma trait em um tipo

Agora que definimos as assinaturas desejadas dos métodos da trait `Summary`, podemos implementá-la nos tipos do nosso agregador de mídia. A [Listagem 10-13](#listagem-10-13) mostra uma implementação da trait `Summary` na struct `NewsArticle` que usa o título, o autor e o local para criar o valor de retorno de `summarize`. Para a struct `SocialPost`, definimos `summarize` como o nome de usuário seguido do texto inteiro da postagem, assumindo que o conteúdo da postagem já está limitado a 280 caracteres.

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, por {} ({})", self.headline, self.author, self.location)
    }
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

<a id="listagem-10-13"></a>

[Listagem 10-13](#listagem-10-13): Implementando a trait `Summary` nos tipos `NewsArticle` e `SocialPost`

Implementar uma trait em um tipo é semelhante a implementar métodos regulares. A diferença é que, após `impl`, colocamos o nome da trait que queremos implementar, depois a palavra-chave `for` e então o nome do tipo para o qual queremos implementar a trait. Dentro do bloco `impl`, colocamos as assinaturas de método que a definição da trait definiu. Em vez de adicionar um ponto e vírgula após cada assinatura, usamos chaves e preenchemos o corpo do método com o comportamento específico que queremos que os métodos da trait tenham para o tipo em questão.

Agora que a biblioteca implementou a trait `Summary` em `NewsArticle` e `SocialPost`, usuários da crate podem chamar os métodos da trait em instâncias de `NewsArticle` e `SocialPost` da mesma forma que chamamos métodos regulares. A única diferença é que o usuário também precisa trazer a trait para o escopo, além dos tipos. Aqui está um exemplo de como uma crate binária poderia usar nossa crate de biblioteca `aggregator`:

**Arquivo: src/main.rs**

```rust
use aggregator::{SocialPost, Summary};

fn main() {
    let post = SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    };

    println!("1 new post: {}", post.summarize());
}
```

Este código imprime `1 new post: horse_ebooks: of course, as you probably already know, people`.

Outras crates que dependem da crate `aggregator` também podem trazer a trait `Summary` para o escopo para implementar `Summary` em seus próprios tipos. Uma restrição a observar é que só podemos implementar uma trait em um tipo se a trait ou o tipo, ou ambos, forem locais à nossa crate. Por exemplo, podemos implementar traits da biblioteca padrão como `Display` em um tipo personalizado como `SocialPost` como parte da funcionalidade da nossa crate `aggregator`, porque o tipo `SocialPost` é local à nossa crate `aggregator`. Também podemos implementar `Summary` em `Vec<T>` na nossa crate `aggregator` porque a trait `Summary` é local à nossa crate `aggregator`.

Mas não podemos implementar traits externas em tipos externos. Por exemplo, não podemos implementar a trait `Display` em `Vec<T>` dentro da nossa crate `aggregator`, porque `Display` e `Vec<T>` estão ambos definidos na biblioteca padrão e não são locais à nossa crate `aggregator`. Esta restrição faz parte de uma propriedade chamada _coerência_ e, mais especificamente, a _regra do órfão_, assim nomeada porque o tipo pai não está presente. Esta regra garante que o código de outras pessoas não possa quebrar o seu e vice-versa. Sem a regra, duas crates poderiam implementar a mesma trait para o mesmo tipo, e o Rust não saberia qual implementação usar.

### Usando implementações padrão

Às vezes é útil ter comportamento padrão para alguns ou todos os métodos em uma trait, em vez de exigir implementações para todos os métodos em todo tipo. Então, ao implementar a trait em um tipo particular, podemos manter ou substituir o comportamento padrão de cada método.

Na [Listagem 10-14](#listagem-10-14), especificamos uma string padrão para o método `summarize` da trait `Summary`, em vez de definir apenas a assinatura do método, como fizemos na [Listagem 10-12](#listagem-10-12).

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

<a id="listagem-10-14"></a>

[Listagem 10-14](#listagem-10-14): Definindo uma trait `Summary` com implementação padrão do método `summarize`

Para usar uma implementação padrão e resumir instâncias de `NewsArticle`, especificamos um bloco `impl` vazio com `impl Summary for NewsArticle {}`.

Embora não estejamos mais definindo o método `summarize` em `NewsArticle` diretamente, fornecemos uma implementação padrão e especificamos que `NewsArticle` implementa a trait `Summary`. Como resultado, ainda podemos chamar o método `summarize` em uma instância de `NewsArticle`, assim:

**Arquivo: src/main.rs**

```rust
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from(
        "The Pittsburgh Penguins once again are the best \
            hockey team in the NHL.",
    ),
};

println!("New article available! {}", article.summarize());
```

Este código imprime `New article available! (Read more...)`.

Criar uma implementação padrão não exige que mudemos nada na implementação de `Summary` em `SocialPost` da [Listagem 10-13](#listagem-10-13). O motivo é que a sintaxe para substituir uma implementação padrão é a mesma da sintaxe para implementar um método de trait que não tem implementação padrão.

Implementações padrão podem chamar outros métodos na mesma trait, mesmo que esses outros métodos não tenham implementação padrão. Assim, uma trait pode fornecer muita funcionalidade útil e exigir apenas que quem implementa especifique uma pequena parte. Por exemplo, poderíamos definir a trait `Summary` para ter um método `summarize_author` cuja implementação é obrigatória, e então definir um método `summarize` com implementação padrão que chama o método `summarize_author`:

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

Para usar esta versão de `Summary`, só precisamos definir `summarize_author` quando implementamos a trait em um tipo:

**Arquivo: src/lib.rs**

```rust
impl Summary for SocialPost {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

Depois de definir `summarize_author`, podemos chamar `summarize` em instâncias da struct `SocialPost`, e a implementação padrão de `summarize` chamará a definição de `summarize_author` que fornecemos. Como implementamos `summarize_author`, a trait `Summary` nos deu o comportamento do método `summarize` sem exigir que escrevêssemos mais código. Veja como fica:

**Arquivo: src/main.rs**

```rust
use aggregator::{self, SocialPost, Summary};

fn main() {
    let post = SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    };

    println!("1 new post: {}", post.summarize());
}
```

Este código imprime `1 new post: (Read more from @horse_ebooks...)`.

Observe que não é possível chamar a implementação padrão a partir de uma implementação substitutiva do mesmo método.

### Usando traits como parâmetros

Agora que você sabe definir e implementar traits, podemos explorar como usá-las para definir funções que aceitam muitos tipos diferentes. Usaremos a trait `Summary` que implementamos nos tipos `NewsArticle` e `SocialPost` da [Listagem 10-13](#listagem-10-13) para definir uma função `notify` que chama o método `summarize` em seu parâmetro `item`, que é de algum tipo que implementa a trait `Summary`. Para isso, usamos a sintaxe `impl Trait`, assim:

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

pub fn notify(item: &impl Summary) {
    println!("Notícia urgente! {}", item.summarize());
}
```

Em vez de um tipo concreto para o parâmetro `item`, especificamos a palavra-chave `impl` e o nome da trait. Este parâmetro aceita qualquer tipo que implemente a trait especificada. No corpo de `notify`, podemos chamar quaisquer métodos em `item` que vêm da trait `Summary`, como `summarize`. Podemos chamar `notify` e passar qualquer instância de `NewsArticle` ou `SocialPost`. Código que chama a função com qualquer outro tipo, como `String` ou `i32`, não compilará, porque esses tipos não implementam `Summary`.

#### Sintaxe de trait bound

A sintaxe `impl Trait` funciona para casos simples, mas é na verdade açúcar sintático para uma forma mais longa conhecida como _trait bound_; parece com isto:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Notícia urgente! {}", item.summarize());
}
```

Esta forma mais longa é equivalente ao exemplo da seção anterior, mas é mais verbosa. Colocamos trait bounds com a declaração do parâmetro de tipo genérico após dois pontos e dentro de colchetes angulares.

A sintaxe `impl Trait` é conveniente e produz código mais conciso em casos simples, enquanto a sintaxe completa de trait bound pode expressar mais complexidade em outros casos. Por exemplo, podemos ter dois parâmetros que implementam `Summary`. Com a sintaxe `impl Trait`, fica assim:

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Usar `impl Trait` é apropriado se quisermos que esta função permita que `item1` e `item2` tenham tipos diferentes (desde que ambos implementem `Summary`). Se quisermos forçar ambos os parâmetros a terem o mesmo tipo, porém, devemos usar um trait bound, assim:

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

O tipo genérico `T` especificado como tipo dos parâmetros `item1` e `item2` restringe a função de modo que o tipo concreto do valor passado como argumento para `item1` e `item2` deve ser o mesmo.

#### Múltiplos trait bounds com a sintaxe `+`

Também podemos especificar mais de um trait bound. Digamos que queremos que `notify` use formatação de exibição além de `summarize` em `item`: especificamos na definição de `notify` que `item` deve implementar `Display` e `Summary`. Podemos fazer isso usando a sintaxe `+`:

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

A sintaxe `+` também é válida com trait bounds em tipos genéricos:

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

Com os dois trait bounds especificados, o corpo de `notify` pode chamar `summarize` e usar `{}` para formatar `item`.

#### Trait bounds mais claros com cláusulas `where`

Usar muitos trait bounds tem suas desvantagens. Cada genérico tem seus próprios trait bounds, então funções com vários parâmetros de tipo genérico podem conter muita informação de trait bound entre o nome da função e sua lista de parâmetros, tornando a assinatura difícil de ler. Por isso, o Rust tem sintaxe alternativa para especificar trait bounds dentro de uma cláusula `where` após a assinatura da função. Em vez de escrever isto:

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

podemos usar uma cláusula `where`, assim:

**Arquivo: src/lib.rs**

```rust
use std::fmt::{Debug, Display};

fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    unimplemented!()
}
```

A assinatura desta função fica menos poluída: o nome da função, a lista de parâmetros e o tipo de retorno ficam próximos, semelhante a uma função sem muitos trait bounds.

### Retornando tipos que implementam traits

Também podemos usar a sintaxe `impl Trait` na posição de retorno para retornar um valor de algum tipo que implementa uma trait, como mostrado aqui:

**Arquivo: src/lib.rs**

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, por {} ({})", self.headline, self.author, self.location)
    }
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn returns_summarizable() -> impl Summary {
    SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    }
}
```

Ao usar `impl Summary` como tipo de retorno, especificamos que a função `returns_summarizable` retorna algum tipo que implementa a trait `Summary` sem nomear o tipo concreto. Neste caso, `returns_summarizable` retorna um `SocialPost`, mas o código que chama esta função não precisa saber disso.

A capacidade de especificar um tipo de retorno apenas pela trait que implementa é especialmente útil no contexto de closures e iterators, que cobriremos no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures). Closures e iterators criam tipos que só o compilador conhece ou tipos muito longos de especificar. A sintaxe `impl Trait` permite especificar concisamente que uma função retorna algum tipo que implementa a trait `Iterator` sem precisar escrever um tipo muito longo.

Porém, você só pode usar `impl Trait` se estiver retornando um único tipo. Por exemplo, este código que retorna `NewsArticle` ou `SocialPost` com o tipo de retorno especificado como `impl Summary` não funcionaria:

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, por {} ({})", self.headline, self.author, self.location)
    }
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Pinguins vencem o campeonato da Stanley Cup!",
            ),
            location: String::from("Pittsburgh, PA, EUA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "Os Pittsburgh Penguins mais uma vez são o melhor \
                 time de hóquei da NHL.",
            ),
        }
    } else {
        SocialPost {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            repost: false,
        }
    }
}
```

Retornar `NewsArticle` ou `SocialPost` não é permitido devido a restrições em torno de como a sintaxe `impl Trait` é implementada no compilador. Cobriremos como escrever uma função com esse comportamento na seção [Usando trait objects para abstrair comportamento compartilhado](/livro/cap18-02-usando-trait-objects-para-abstrair-comportamento-compartilhado) do Capítulo 18.

### Usando trait bounds para implementar métodos condicionalmente

Usando um trait bound com um bloco `impl` que usa parâmetros de tipo genérico, podemos implementar métodos condicionalmente para tipos que implementam as traits especificadas. Por exemplo, o tipo `Pair<T>` na [Listagem 10-15](#listagem-10-15) sempre implementa a função `new` para retornar uma nova instância de `Pair<T>` (lembre-se da seção [Sintaxe de métodos](/livro/cap05-03-sintaxe-de-metodos) do Capítulo 5 de que `Self` é um alias de tipo para o tipo do bloco `impl`, que neste caso é `Pair<T>`). Mas no próximo bloco `impl`, `Pair<T>` só implementa o método `cmp_display` se seu tipo interno `T` implementar a trait `PartialOrd`, que habilita comparação, _e_ a trait `Display`, que habilita impressão.

**Arquivo: src/lib.rs**

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("O maior membro é x = {}", self.x);
        } else {
            println!("O maior membro é y = {}", self.y);
        }
    }
}
```

<a id="listagem-10-15"></a>

[Listagem 10-15](#listagem-10-15): Implementando métodos condicionalmente em um tipo genérico dependendo de trait bounds

Também podemos implementar condicionalmente uma trait para qualquer tipo que implemente outra trait. Implementações de uma trait em qualquer tipo que satisfaça os trait bounds são chamadas de _implementações abrangentes_ e são usadas extensivamente na biblioteca padrão do Rust. Por exemplo, a biblioteca padrão implementa a trait `ToString` em qualquer tipo que implemente a trait `Display`. O bloco `impl` na biblioteca padrão parece semelhante a este código:

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

Como a biblioteca padrão tem esta implementação abrangente, podemos chamar o método `to_string` definido pela trait `ToString` em qualquer tipo que implemente `Display`. Por exemplo, podemos converter inteiros em seus valores `String` correspondentes assim, porque inteiros implementam `Display`:

```rust
let s = 3.to_string();
```

Implementações abrangentes aparecem na documentação da trait na seção “Implementors” (Implementadores).

Traits e trait bounds nos permitem escrever código que usa parâmetros de tipo genérico para reduzir duplicação, mas também especificar ao compilador que queremos que o tipo genérico tenha comportamento particular. O compilador pode então usar a informação de trait bound para verificar se todos os tipos concretos usados com nosso código fornecem o comportamento correto. Em linguagens com tipagem dinâmica, obteríamos um erro em tempo de execução se chamássemos um método em um tipo que não define o método. Mas o Rust move esses erros para tempo de compilação, de modo que somos forçados a corrigir os problemas antes mesmo de o código poder executar. Além disso, não precisamos escrever código que verifica comportamento em tempo de execução, porque já verificamos em tempo de compilação. Isso melhora o desempenho sem abrir mão da flexibilidade dos generics.
