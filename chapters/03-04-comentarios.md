---
chapter_code: 03-04
slug: comentarios
---

# Comentários

Todo programador busca escrever código fácil de entender, mas às vezes uma explicação extra é necessária. Nesses casos, os programadores deixam _comentários_ no código-fonte — trechos que o compilador ignora, mas que são úteis para pessoas que estão lendo o código.

Aqui está um comentário simples:

```rust
// hello, world
```

Em Rust, o estilo idiomático de comentários começa com duas barras (`//`), e o comentário continua até o final da linha. Para comentários que se estendem por mais de uma linha, é necessário incluir `//` em cada linha, como neste exemplo:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

Os comentários também podem ser colocados no final de linhas que contêm código:

**Arquivo: src/main.rs**

```rust
fn main() {
    let lucky_number = 7; // I'm feeling lucky today
}
```

No entanto, é mais comum vê-los usados neste formato, com o comentário em uma linha separada, logo acima do código que ele está explicando:

**Arquivo: src/main.rs**

```rust
fn main() {
    // I'm feeling lucky today
    let lucky_number = 7;
}
```

O Rust também possui outro tipo de comentário, chamado comentários de documentação, que serão discutidos na seção "Publicando um crate no crates.io" do Capítulo 14.