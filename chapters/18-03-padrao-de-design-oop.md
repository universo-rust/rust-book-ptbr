---
title: "Implementando um padrão de design orientado a objetos"
chapter_code: 18-03
slug: implementando-um-padrao-de-design-orientado-a-objetos
---

# Implementando um padrão de design orientado a objetos

O _padrão state_ é um padrão de design orientado a objetos. A ideia central desse padrão é definir um conjunto de estados que um valor pode ter internamente. Esses estados são representados por um conjunto de _objetos de estado_, e o comportamento do valor muda conforme o estado atual. Vamos trabalhar com o exemplo de uma struct de post de blog que tem um campo para guardar seu estado, que será um objeto de estado entre "rascunho", "em revisão" e "publicado".

Os objetos de estado compartilham funcionalidade. Em Rust, é claro, usamos structs e traits em vez de objetos e herança. Cada objeto de estado é responsável por seu próprio comportamento e por controlar quando deve se transformar em outro estado. O valor que guarda um objeto de estado não sabe nada sobre os comportamentos diferentes dos estados nem sobre quando as transições entre estados devem acontecer.

A vantagem de usar o padrão state é que, quando os requisitos de negócio do programa mudam, não precisamos alterar o código do valor que guarda o estado nem o código que usa esse valor. Precisamos apenas atualizar o código dentro de um dos objetos de estado para mudar suas regras ou talvez adicionar novos objetos de estado.

Primeiro, vamos implementar o padrão state de uma forma mais tradicional em programação orientada a objetos. Depois, usaremos uma abordagem um pouco mais natural em Rust. Vamos implementar aos poucos um fluxo de trabalho para posts de blog usando o padrão state.

A funcionalidade final será assim:

1. Um post de blog começa como um rascunho vazio.
2. Quando o rascunho está pronto, uma revisão do post é solicitada.
3. Quando o post é aprovado, ele é publicado.
4. Apenas posts publicados retornam conteúdo para impressão, para que posts não aprovados não sejam publicados por acidente.

Qualquer outra tentativa de alteração em um post não deve ter efeito. Por exemplo, se tentarmos aprovar um post em rascunho antes de solicitar revisão, o post deve continuar sendo um rascunho não publicado.

## Tentativa no estilo tradicional orientado a objetos

Há infinitas maneiras de estruturar código para resolver o mesmo problema, cada uma com tradeoffs diferentes. A implementação desta seção segue um estilo mais tradicional de programação orientada a objetos, que é possível escrever em Rust, mas não aproveita algumas das forças da linguagem. Mais adiante, demonstraremos uma solução diferente, que ainda usa o padrão de design orientado a objetos, mas é estruturada de uma forma que pode parecer menos familiar para quem vem de linguagens orientadas a objetos. Vamos comparar as duas soluções para sentir os tradeoffs de projetar código Rust de modo diferente do código em outras linguagens.

A Listagem 18-11 mostra esse fluxo de trabalho em forma de código. Este é um exemplo de uso da API que implementaremos em um crate de biblioteca chamado `blog`. O código ainda não compila porque ainda não implementamos o crate `blog`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<a id="listagem-18-11"></a>

[Listagem 18-11](#listagem-18-11): Código que demonstra o comportamento desejado do crate `blog`

Queremos permitir que a pessoa usuária crie um novo post de blog em rascunho com `Post::new`. Também queremos permitir que texto seja adicionado ao post. Se tentarmos obter o conteúdo imediatamente, antes da aprovação, não devemos receber nenhum texto, porque o post ainda é um rascunho. Colocamos `assert_eq!` no código para fins de demonstração. Um excelente teste unitário para isso seria verificar que um post em rascunho retorna uma string vazia a partir do método `content`, mas não vamos escrever testes para este exemplo.

Em seguida, queremos permitir que uma revisão do post seja solicitada, e queremos que `content` continue retornando uma string vazia enquanto o post espera revisão. Quando o post receber aprovação, ele deve ser publicado, ou seja, o texto do post será retornado quando `content` for chamado.

Observe que o único tipo com que interagimos a partir do crate é `Post`. Esse tipo usará o padrão state e guardará um valor que será um entre três objetos de estado, representando os estados possíveis de um post: rascunho, revisão ou publicado. A mudança de um estado para outro será gerenciada internamente pelo tipo `Post`. Os estados mudam em resposta aos métodos chamados por quem usa nossa biblioteca em uma instância de `Post`, mas essas pessoas não precisam gerenciar as mudanças de estado diretamente. Além disso, não conseguem cometer erros com os estados, como publicar um post antes que ele seja revisado.

### Definindo `Post` e criando uma nova instância

Vamos começar a implementação da biblioteca! Sabemos que precisamos de uma struct pública `Post` que guarda algum conteúdo, então começaremos com a definição da struct e uma função associada pública `new` para criar uma instância de `Post`, como mostra a Listagem 18-12. Também criaremos uma trait privada `State`, que definirá o comportamento que todos os objetos de estado de um `Post` devem ter.

Então, `Post` guardará um trait object de `Box<dyn State>` dentro de um `Option<T>` em um campo privado chamado `state`, para guardar o objeto de estado. Daqui a pouco você verá por que o `Option<T>` é necessário.

**Arquivo: src/lib.rs**

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

<a id="listagem-18-12"></a>

[Listagem 18-12](#listagem-18-12): Definição de `Post`, da função `new`, da trait `State` e da struct `Draft`

A trait `State` define o comportamento compartilhado pelos diferentes estados de um post. Os objetos de estado são `Draft`, `PendingReview` e `Published`, e todos implementarão a trait `State`. Por enquanto, a trait não tem métodos, e começaremos definindo apenas o estado `Draft`, porque esse é o estado inicial de um post.

Quando criamos um novo `Post`, definimos seu campo `state` como um valor `Some` que guarda um `Box`. Esse `Box` aponta para uma nova instância da struct `Draft`. Isso garante que toda nova instância de `Post` comece como rascunho. Como o campo `state` de `Post` é privado, não há como criar um `Post` em qualquer outro estado. Na função `Post::new`, também definimos o campo `content` como uma `String` nova e vazia.

### Armazenando o texto do post

Vimos na Listagem 18-11 que queremos poder chamar um método chamado `add_text` e passar a ele um `&str`, que será adicionado como conteúdo textual do post de blog. Implementamos isso como um método, em vez de expor o campo `content` como `pub`, para que mais tarde possamos implementar um método que controle como os dados do campo `content` são lidos. O método `add_text` é bastante direto, então vamos adicioná-lo ao bloco `impl Post`, como na Listagem 18-13.

**Arquivo: src/lib.rs**

```rust
pub fn add_text(&mut self, text: &str) {
    self.content.push_str(text);
}
```

<a id="listagem-18-13"></a>

[Listagem 18-13](#listagem-18-13): Um método `add_text` para adicionar texto ao campo `content` de um post

O método `add_text` recebe uma referência mutável para `self` porque estamos alterando a instância de `Post` em que `add_text` é chamado. Em seguida, chamamos `push_str` na `String` em `content` e passamos o argumento `text` para acrescentá-lo ao conteúdo salvo. Esse comportamento não depende do estado do post, então não faz parte do padrão state. O método `add_text` não interage com o campo `state`, mas faz parte do comportamento que queremos oferecer.

### Garantindo que o conteúdo de um post em rascunho esteja vazio

Mesmo depois de chamarmos `add_text` e adicionarmos algum conteúdo ao post, ainda queremos que o método `content` retorne uma string slice vazia, porque o post continua no estado de rascunho, como mostra o primeiro `assert_eq!` na Listagem 18-11. Por enquanto, vamos implementar `content` da maneira mais simples que satisfaz esse requisito: sempre retornando uma string slice vazia. Mudaremos isso depois, quando implementarmos a capacidade de mudar o estado do post para que ele possa ser publicado. Até aqui, posts só podem estar no estado de rascunho, então o conteúdo do post deve sempre estar vazio. A Listagem 18-14 mostra essa implementação temporária.

**Arquivo: src/lib.rs**

```rust
pub fn content(&self) -> &str {
    ""
}
```

<a id="listagem-18-14"></a>

[Listagem 18-14](#listagem-18-14): Implementação provisória de `content` que sempre retorna uma string slice vazia

Com esse método `content` adicionado, tudo na Listagem 18-11 até o primeiro `assert_eq!` funciona como esperado.

### Solicitando uma revisão, o que muda o estado do post

Agora precisamos adicionar a funcionalidade de solicitar revisão de um post, o que deve mudar seu estado de `Draft` para `PendingReview`. A Listagem 18-15 mostra esse código.

**Arquivo: src/lib.rs**

```rust
pub fn request_review(&mut self) {
    if let Some(s) = self.state.take() {
        self.state = Some(s.request_review())
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

<a id="listagem-18-15"></a>

[Listagem 18-15](#listagem-18-15): Implementando o método `request_review` em `Post` e na trait `State`

Damos a `Post` um método público chamado `request_review`, que recebe uma referência mutável para `self`. Então, chamamos um método interno `request_review` no estado atual de `Post`; esse segundo `request_review` consome o estado atual e retorna um novo estado.

Adicionamos o método `request_review` à trait `State`; todos os tipos que implementam essa trait agora precisarão implementar esse método. Repare que, em vez de `self`, `&self` ou `&mut self` como primeiro parâmetro, usamos `self: Box<Self>`. Essa sintaxe significa que o método só é válido quando chamado em um `Box` que guarda o tipo. Ela assume ownership de `Box<Self>`, invalidando o estado antigo para que o valor de estado de `Post` possa se transformar em um novo estado.

Para consumir o estado antigo, o método `request_review` precisa assumir ownership do valor de estado. É aqui que entra o `Option` no campo `state` de `Post`: chamamos o método `take` para retirar o valor `Some` do campo `state` e deixar um `None` no lugar, porque Rust não permite campos não preenchidos em structs. Isso nos permite mover o valor de `state` para fora de `Post`, em vez de apenas emprestá-lo. Depois, definimos o valor de `state` do post como o resultado dessa operação.

Precisamos definir `state` como `None` temporariamente, em vez de fazer algo como `self.state = self.state.request_review();`, para conseguir ownership do valor de `state`. Isso garante que `Post` não possa usar o valor antigo de `state` depois de o transformarmos em um novo estado.

O método `request_review` em `Draft` retorna uma nova instância encaixotada da nova struct `PendingReview`, que representa o estado em que um post está aguardando revisão. A struct `PendingReview` também implementa o método `request_review`, mas não faz nenhuma transformação. Em vez disso, retorna a si mesma, porque, se solicitarmos revisão de um post que já está em `PendingReview`, ele deve permanecer em `PendingReview`.

Agora começamos a ver as vantagens do padrão state: o método `request_review` em `Post` é o mesmo independentemente do valor de `state`. Cada estado é responsável por suas próprias regras.

Vamos deixar o método `content` em `Post` como está, retornando uma string slice vazia. Agora podemos ter um `Post` tanto no estado `PendingReview` quanto no estado `Draft`, mas queremos o mesmo comportamento para `content` em `PendingReview`. A Listagem 18-11 agora funciona até o segundo `assert_eq!`.

### Adicionando `approve` para mudar o comportamento de `content`

O método `approve` será parecido com `request_review`: ele definirá `state` como o valor que o estado atual diz que deve assumir quando esse estado é aprovado, como mostra a Listagem 18-16.

**Arquivo: src/lib.rs**

```rust
pub fn approve(&mut self) {
    if let Some(s) = self.state.take() {
        self.state = Some(s.approve())
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

<a id="listagem-18-16"></a>

[Listagem 18-16](#listagem-18-16): Implementando o método `approve` em `Post` e na trait `State`

Adicionamos o método `approve` à trait `State` e acrescentamos uma nova struct que implementa `State`: o estado `Published`.

De forma semelhante ao que acontece com `request_review` em `PendingReview`, se chamarmos `approve` em um `Draft`, nada mudará, porque `approve` retorna `self`. Quando chamamos `approve` em `PendingReview`, ele retorna uma nova instância encaixotada da struct `Published`. A struct `Published` implementa a trait `State` e, tanto para `request_review` quanto para `approve`, retorna a si mesma, porque nesses casos o post deve permanecer no estado `Published`.

Agora precisamos atualizar o método `content` em `Post`. Queremos que o valor retornado por `content` dependa do estado atual de `Post`, então faremos `Post` delegar para um método `content` definido em seu `state`, como mostra a Listagem 18-17.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn content(&self) -> &str {
    self.state.as_ref().unwrap().content(self)
}
```

<a id="listagem-18-17"></a>

[Listagem 18-17](#listagem-18-17): Atualizando o método `content` em `Post` para delegar a um método `content` em `State`

Como o objetivo é manter todas essas regras dentro das structs que implementam `State`, chamamos um método `content` no valor em `state` e passamos a instância do post, isto é, `self`, como argumento. Depois, retornamos o valor retornado pelo método `content` do valor de estado.

Chamamos o método `as_ref` no `Option` porque queremos uma referência para o valor dentro do `Option`, não ownership desse valor. Como `state` é um `Option<Box<dyn State>>`, chamar `as_ref` retorna um `Option<&Box<dyn State>>`. Se não chamássemos `as_ref`, receberíamos um erro porque não podemos mover `state` para fora do `&self` emprestado no parâmetro da função.

Em seguida, chamamos `unwrap`, que sabemos que nunca entrará em pânico porque os métodos de `Post` garantem que `state` sempre conterá um valor `Some` quando eles terminarem. Esse é um dos casos discutidos na seção ["Quando você sabe mais que o compilador"](/livro/cap09-03-panic-ou-nao-panic#quando-voce-sabe-mais-que-o-compilador), do Capítulo 9, quando sabemos que um valor `None` nunca é possível, mesmo que o compilador não consiga entender isso.

Nesse ponto, quando chamamos `content` no `&Box<dyn State>`, a coerção de deref entra em ação no `&` e no `Box`, de modo que o método `content` acaba sendo chamado no tipo que implementa a trait `State`. Isso significa que precisamos adicionar `content` à definição da trait `State`; é ali que colocaremos a lógica sobre qual conteúdo retornar dependendo do estado, como mostra a Listagem 18-18.

**Arquivo: src/lib.rs**

```rust
trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;

    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

<a id="listagem-18-18"></a>

[Listagem 18-18](#listagem-18-18): Adicionando o método `content` à trait `State`

Adicionamos uma implementação padrão para o método `content` que retorna uma string slice vazia. Isso significa que não precisamos implementar `content` nas structs `Draft` e `PendingReview`. A struct `Published` sobrescreve o método `content` e retorna o valor em `post.content`. Embora seja conveniente, deixar o método `content` em `State` decidir o conteúdo de `Post` mistura um pouco as responsabilidades de `State` e de `Post`.

Observe que precisamos de anotações de lifetime nesse método, como discutimos no Capítulo 10. Recebemos uma referência para `post` como argumento e retornamos uma referência para parte desse `post`, então o lifetime da referência retornada está ligado ao lifetime do argumento `post`.

E pronto: toda a Listagem 18-11 agora funciona! Implementamos o padrão state com as regras do fluxo de trabalho do post de blog. A lógica relacionada às regras fica nos objetos de estado em vez de ficar espalhada por `Post`.

> ### Por que não um enum?
>
> Talvez você esteja se perguntando por que não usamos um enum com os diferentes estados possíveis de post como variantes. Essa certamente é uma solução possível; experimente e compare os resultados para ver qual prefere! Uma desvantagem de usar um enum é que todo lugar que verifica o valor do enum precisará de uma expressão `match`, ou algo semelhante, para tratar todas as variantes possíveis. Isso pode ficar mais repetitivo do que a solução com trait objects.

### Avaliando o padrão state

Mostramos que Rust é capaz de implementar o padrão state orientado a objetos para encapsular os diferentes tipos de comportamento que um post deve ter em cada estado. Os métodos de `Post` não sabem nada sobre esses comportamentos variados. Pela forma como organizamos o código, precisamos olhar em apenas um lugar para saber as diferentes maneiras como um post publicado pode se comportar: a implementação da trait `State` para a struct `Published`.

Se criássemos uma implementação alternativa sem o padrão state, poderíamos usar expressões `match` nos métodos de `Post`, ou até mesmo no código de `main`, para verificar o estado do post e mudar o comportamento nesses lugares. Isso significaria que precisaríamos olhar em vários lugares para entender todas as implicações de um post estar no estado publicado.

Com o padrão state, os métodos de `Post` e os lugares em que usamos `Post` não precisam de expressões `match`; para adicionar um novo estado, bastaria adicionar uma nova struct e implementar os métodos da trait para essa struct em um único lugar.

A implementação com o padrão state é fácil de estender com mais funcionalidades. Para ver como a manutenção do código fica simples com esse padrão, experimente algumas destas sugestões:

- Adicione um método `reject` que mude o estado do post de `PendingReview` de volta para `Draft`.
- Exija duas chamadas a `approve` antes que o estado possa mudar para `Published`.
- Permita que usuários adicionem conteúdo de texto apenas quando o post estiver no estado `Draft`. Dica: faça o objeto de estado ser responsável pelo que pode mudar em relação ao conteúdo, mas não responsável por modificar o `Post`.

Uma desvantagem do padrão state é que, como os estados implementam as transições entre estados, alguns estados ficam acoplados entre si. Se adicionarmos outro estado entre `PendingReview` e `Published`, como `Scheduled`, teríamos que alterar o código de `PendingReview` para transicionar para `Scheduled`. Daria menos trabalho se `PendingReview` não precisasse mudar com a adição de um novo estado, mas isso significaria escolher outro padrão de design.

Outra desvantagem é que duplicamos alguma lógica. Para eliminar parte da duplicação, poderíamos tentar criar implementações padrão dos métodos `request_review` e `approve` na trait `State` que retornassem `self`. No entanto, isso não funcionaria: ao usar `State` como trait object, a trait não sabe qual será exatamente o tipo concreto de `self`, então o tipo de retorno não é conhecido em tempo de compilação. Essa é uma das regras de compatibilidade com `dyn` mencionadas anteriormente.

Outra duplicação aparece nas implementações parecidas dos métodos `request_review` e `approve` em `Post`. Ambos usam `Option::take` com o campo `state` de `Post` e, se `state` for `Some`, delegam para a implementação do mesmo método no valor embrulhado e definem o novo valor do campo `state` como o resultado. Se tivéssemos muitos métodos em `Post` seguindo esse padrão, poderíamos considerar definir uma macro para eliminar a repetição; veja a seção ["Macros"](/livro/cap20-05-macros#macros), no Capítulo 20.

Ao implementar o padrão state exatamente como ele é definido para linguagens orientadas a objetos, não aproveitamos os pontos fortes de Rust tanto quanto poderíamos. Vamos ver algumas mudanças que podemos fazer no crate `blog` para transformar estados e transições inválidos em erros de compilação.

## Codificando estados e comportamento como tipos

Vamos mostrar como repensar o padrão state para obter um conjunto diferente de tradeoffs. Em vez de encapsular completamente os estados e transições para que o código externo não saiba nada sobre eles, vamos codificar os estados em tipos diferentes. Como consequência, o sistema de tipos de Rust impedirá tentativas de usar posts em rascunho onde apenas posts publicados são permitidos, emitindo um erro de compilação.

Vamos considerar a primeira parte de `main` na Listagem 18-11:

**Arquivo: src/main.rs**

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
}
```

Ainda permitiremos a criação de novos posts no estado de rascunho usando `Post::new` e a possibilidade de adicionar texto ao conteúdo do post. Porém, em vez de ter um método `content` em um post em rascunho que retorna uma string vazia, faremos com que posts em rascunho não tenham o método `content`. Dessa forma, se tentarmos obter o conteúdo de um rascunho, receberemos um erro de compilação dizendo que o método não existe. Como resultado, será impossível exibir acidentalmente conteúdo de rascunho em produção, porque esse código nem sequer compilará. A Listagem 18-19 mostra a definição de uma struct `Post`, de uma struct `DraftPost` e os métodos de cada uma.

**Arquivo: src/lib.rs**

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

<a id="listagem-18-19"></a>

[Listagem 18-19](#listagem-18-19): Um `Post` com um método `content` e um `DraftPost` sem método `content`

As structs `Post` e `DraftPost` têm um campo privado `content` que armazena o texto do post de blog. As structs não têm mais o campo `state`, porque estamos movendo a representação do estado para os próprios tipos das structs. A struct `Post` representará um post publicado e terá um método `content` que retorna o conteúdo.

Ainda temos uma função `Post::new`, mas, em vez de retornar uma instância de `Post`, ela retorna uma instância de `DraftPost`. Como `content` é privado e não há nenhuma função que retorne `Post`, não é possível criar uma instância de `Post` neste momento.

A struct `DraftPost` tem um método `add_text`, então podemos adicionar texto a `content` como antes. Mas repare que `DraftPost` não tem um método `content` definido! Agora o programa garante que todos os posts começam como rascunhos e que rascunhos não têm seu conteúdo disponível para exibição. Qualquer tentativa de contornar essas restrições resultará em erro de compilação.

### Implementando transições como transformações de tipos

Então, como obtemos um post publicado? Queremos impor a regra de que um post em rascunho precisa ser revisado e aprovado antes de ser publicado. Um post em estado de revisão pendente ainda não deve exibir conteúdo. Vamos implementar essas restrições adicionando outra struct, `PendingReviewPost`, definindo o método `request_review` em `DraftPost` para retornar um `PendingReviewPost` e definindo um método `approve` em `PendingReviewPost` para retornar um `Post`, como mostra a Listagem 18-20.

**Arquivo: src/lib.rs**

```rust
impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

<a id="listagem-18-20"></a>

[Listagem 18-20](#listagem-18-20): Um `PendingReviewPost` criado chamando `request_review` em `DraftPost` e um método `approve` que transforma `PendingReviewPost` em um `Post` publicado

Os métodos `request_review` e `approve` assumem ownership de `self`, consumindo as instâncias de `DraftPost` e `PendingReviewPost` e transformando-as em um `PendingReviewPost` e em um `Post` publicado, respectivamente. Dessa forma, não ficam instâncias antigas de `DraftPost` depois que chamamos `request_review`, e assim por diante. A struct `PendingReviewPost` não tem um método `content` definido, então tentar ler seu conteúdo resulta em erro de compilação, como acontece com `DraftPost`. Como a única forma de obter uma instância publicada de `Post`, que possui o método `content`, é chamar `approve` em um `PendingReviewPost`, e a única forma de obter um `PendingReviewPost` é chamar `request_review` em um `DraftPost`, codificamos o fluxo de trabalho do post de blog no sistema de tipos.

Mas também precisamos fazer pequenas alterações em `main`. Os métodos `request_review` e `approve` retornam novas instâncias em vez de modificar a struct em que são chamados, então precisamos adicionar mais atribuições com sombreamento, `let post =`, para salvar as instâncias retornadas. Também não podemos manter as asserções de que os conteúdos dos posts em rascunho e em revisão pendente são strings vazias, nem precisamos delas: não conseguimos compilar código que tente usar o conteúdo de posts nesses estados. O código atualizado de `main` aparece na Listagem 18-21.

**Arquivo: src/main.rs**

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<a id="listagem-18-21"></a>

[Listagem 18-21](#listagem-18-21): Modificações em `main` para usar a nova implementação do fluxo de trabalho de posts de blog

As mudanças em `main` para reatribuir `post` significam que essa implementação já não segue exatamente o padrão state orientado a objetos: as transformações entre estados não ficam mais totalmente encapsuladas dentro da implementação de `Post`. Porém, ganhamos algo importante: estados inválidos agora são impossíveis por causa do sistema de tipos e da verificação feita em tempo de compilação. Isso garante que certos bugs, como exibir o conteúdo de um post não publicado, sejam descobertos antes de chegarem à produção.

Experimente no crate `blog`, depois da Listagem 18-21, as tarefas sugeridas no início desta seção para ver o que você acha do design desta versão do código. Note que algumas dessas tarefas talvez já estejam resolvidas por esse design.

Vimos que, embora Rust seja capaz de implementar padrões de design orientados a objetos, outros padrões, como codificar estado no sistema de tipos, também estão disponíveis em Rust. Esses padrões têm tradeoffs diferentes. Mesmo que você conheça muito bem padrões orientados a objetos, repensar o problema para aproveitar os recursos de Rust pode trazer benefícios, como prevenir alguns bugs em tempo de compilação. Padrões orientados a objetos nem sempre serão a melhor solução em Rust por causa de certos recursos, como ownership, que linguagens orientadas a objetos não têm.

## Resumo

Independentemente de você considerar Rust uma linguagem orientada a objetos depois de ler este capítulo, agora sabe que pode usar trait objects para obter alguns recursos orientados a objetos em Rust. Despacho dinâmico pode dar flexibilidade ao seu código em troca de um pouco de desempenho em tempo de execução. Você pode usar essa flexibilidade para implementar padrões orientados a objetos que ajudem na manutenção do código. Rust também tem outros recursos, como ownership, que linguagens orientadas a objetos não têm. Um padrão orientado a objetos nem sempre será a melhor forma de aproveitar os pontos fortes de Rust, mas é uma opção disponível.

A seguir, vamos estudar padrões, outro recurso de Rust que oferece muita flexibilidade. Já os vimos brevemente ao longo do livro, mas ainda não exploramos todo o seu potencial. Vamos lá!
