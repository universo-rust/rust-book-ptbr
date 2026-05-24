---
title: "Implementando um padrão de design orientado a objetos"
chapter_code: 18-03
slug: implementando-um-padrao-de-design-orientado-a-objetos
---

# Implementando um Padrão de Design Orientado a Objetos

O _padrão state_ é um padrão de design orientado a objetos. A ideia é definir estados internos possíveis para um valor, representados por _objetos de estado_; o comportamento muda conforme o estado. Trabalharemos com um post de blog cuja struct guarda estado entre “rascunho”, “revisão” e “publicado”.

Em Rust usamos structs e traits em vez de objetos e herança. Cada estado cuida do próprio comportamento e de quando mudar para outro. O valor que guarda o estado não conhece os detalhes de cada estado.

A vantagem: quando requisitos mudam, não precisamos alterar quem guarda o estado nem quem usa o valor — só o objeto de estado afetado.

Primeiro implementamos o padrão de forma mais tradicional em POO; depois uma abordagem mais natural em Rust.

O fluxo final:

1. Post começa como rascunho vazio.
2. Rascunho pronto → solicita revisão.
3. Aprovado → publicado.
4. Só posts publicados retornam conteúdo para impressão.

Outras mudanças não têm efeito (aprovar rascunho sem revisão mantém rascunho não publicado).

## Tentativa no estilo tradicional orientado a objetos

Há infinitas formas de estruturar o mesmo problema. Esta seção é mais tradicional em POO — possível em Rust, mas sem aproveitar tudo. Depois comparamos com outra solução.

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

`Post::new` cria rascunho; `add_text` adiciona texto; antes da aprovação `content` retorna vazio. Depois revisão e aprovação, `content` retorna o texto. Só interagimos com `Post`, que usa o padrão state internamente — usuários não gerenciam estados nem cometem erros como publicar sem revisar.

### Definindo `Post` e criando nova instância

Começamos com `Post` pública, trait privada `State` e `state: Option<Box<dyn State>>` (o `Option` explicado em breve).

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

[Listagem 18-12](#listagem-18-12): `Post`, `new`, trait `State` e struct `Draft`

Estados `Draft`, `PendingReview` e `Published` implementarão `State`. Por ora só `Draft` — estado inicial. `state` privado impede criar `Post` em outro estado.

### Armazenando o texto do post

**Arquivo: src/lib.rs**

```rust
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
```

<a id="listagem-18-13"></a>

[Listagem 18-13](#listagem-18-13): `add_text` para adicionar texto ao `content`

Não depende do estado; não faz parte do padrão state, mas faz parte do comportamento desejado.

### Garantindo que rascunho tenha conteúdo vazio

**Arquivo: src/lib.rs**

```rust
    pub fn content(&self) -> &str {
        ""
    }
```

<a id="listagem-18-14"></a>

[Listagem 18-14](#listagem-18-14): Implementação provisória de `content` que sempre retorna string vazia

Até o primeiro `assert_eq!` da Listagem 18-11 funciona.

### Solicitar revisão, mudando o estado

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

[Listagem 18-15](#listagem-18-15): `request_review` em `Post` e na trait `State`

`Post::request_review` delega ao estado atual. Na trait, `self: Box<Self>` consome o estado (ownership do `Box`). `take` no `Option` move o estado para transformá-lo — Rust não permite campo vazio na struct.

`Draft` vira `PendingReview`; `PendingReview` em revisão retorna a si mesmo.

`request_review` em `Post` é igual qualquer que seja o estado; cada estado tem suas regras. Até o segundo `assert_eq!` da Listagem 18-11 funciona.

### Adicionando `approve` para mudar o comportamento de `content`

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

[Listagem 18-16](#listagem-18-16): `approve` em `Post` e na trait `State`

`approve` em `Draft` não faz nada; em `PendingReview` vira `Published`; `Published` permanece.

Atualizamos `content` para delegar ao estado:

**Arquivo: src/lib.rs**

```rust
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
```

<a id="listagem-18-17"></a>

[Listagem 18-17](#listagem-18-17): `content` em `Post` delega a `content` em `State`

`as_ref` no `Option` porque queremos referência, não ownership. `unwrap` é seguro aqui: métodos de `Post` garantem `Some` — caso de Quando você tem mais informações que o compilador no Capítulo 9.

Deref coercion leva à implementação correta de `State`. Adicionamos `content` à trait (Listagem 18-18).

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

[Listagem 18-18](#listagem-18-18): Método `content` na trait `State`

Implementação padrão retorna `""`; `Draft` e `PendingReview` não precisam sobrescrever. `Published` retorna `post.content`. Anotações de lifetime como no Capítulo 10: referência retornada ligada à vida de `post`.

A Listagem 18-11 completa funciona. Regras do fluxo ficam nos objetos de estado, não espalhadas em `Post`.

> ### Por que não um enum?
>
> Poderíamos usar enum com variantes de estado. Experimente e compare! Desvantagem: cada verificação exige `match` em todo lugar — pode ser mais repetitivo que trait objects.

### Avaliando o padrão state

Rust implementa o padrão state para encapsular comportamento por estado. Métodos de `Post` não conhecem detalhes; regras de post publicado estão em `Published`.

Sem o padrão, `match` em `Post` ou em `main` espalharia implicações. Com o padrão, sem `match` em quem usa `Post`; novo estado = nova struct + implementação da trait num lugar.

Fácil estender (tente: `reject` de `PendingReview` para `Draft`; dois `approve` para publicar; `add_text` só em `Draft`).

Desvantagens: estados acoplados nas transições (novo estado entre revisão e publicado exige mudar `PendingReview`). Lógica duplicada em `request_review`/`approve` de `Post` — macro no Capítulo 20 em Macros poderia ajudar.

Implementação literal de POO não usa Rust ao máximo. Vamos codificar estados nos tipos para erros em compile time.

### Codificando estados e comportamento como tipos

Em vez de esconder estados, codificamos em tipos diferentes. O compilador impede usar rascunho onde só publicado é permitido.

Trecho inicial da Listagem 18-11: `Post::new` e `add_text` permanecem; rascunho não terá `content` — tentar chamar dá erro de compilação.

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

[Listagem 18-19](#listagem-18-19): `Post` com `content`, `DraftPost` sem `content`

`Post::new` retorna `DraftPost`; sem função que retorne `Post`, não criamos publicado diretamente. Posts começam rascunho; rascunho não exibe conteúdo.

### Implementando transições como transformações de tipos

Rascunho precisa revisão e aprovação antes de publicar. `PendingReviewPost` sem `content`; `request_review` em `DraftPost` → `PendingReviewPost`; `approve` → `Post` (Listagem 18-20).

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

[Listagem 18-20](#listagem-18-20): `PendingReviewPost`, `request_review` e `approve`

Métodos consomem `self` — não sobram `DraftPost` após `request_review`. Só `approve` em `PendingReviewPost` produz `Post` com `content`; só `request_review` em `DraftPost` produz `PendingReviewPost`. Fluxo no sistema de tipos.

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

[Listagem 18-21](#listagem-18-21): `main` com a nova implementação do fluxo do blog

Reatribuições com `let post =` porque métodos retornam novas instâncias. Sem asserts de conteúdo vazio em rascunho/revisão — não compila tentar ler conteúdo nesses estados.

Transições não ficam totalmente encapsuladas em `Post`, mas estados inválidos são impossíveis em compile time — bugs como exibir rascunho em produção não chegam a compilar.

Experimente as tarefas sugeridas nesta versão; algumas já podem estar resolvidas pelo design.

Rust pode implementar POO, mas codificar estado nos tipos também está disponível, com trade-offs diferentes. Repensar o problema com ownership do Rust pode prevenir bugs em compile time.

## Resumo

Rust pode não ser POO para todos, mas trait objects trazem recursos orientados a objetos. Despacho dinâmico dá flexibilidade por custo de runtime. Ownership e outros recursos permitem opções que POO pura não tem.

A seguir, padrões — outro recurso flexível do Rust que vimos ao longo do livro e ainda não em plenitude.
