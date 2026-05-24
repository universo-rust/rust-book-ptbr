---
title: "Palavras-chave"
chapter_code: 22-01
slug: palavras-chave
---

# Apêndice A: Palavras-chave

As listas a seguir contêm palavras-chave reservadas para uso atual ou futuro pela linguagem Rust. Por isso, elas não podem ser usadas como identificadores (exceto como identificadores brutos, como discutimos na seção [“Identificadores brutos”](#identificadores-brutos)). _Identificadores_ são nomes de funções, variáveis, parâmetros, campos de struct, módulos, crates, constantes, macros, valores estáticos, atributos, tipos, traits ou lifetimes.

### Palavras-chave atualmente em uso

A seguir está uma lista de palavras-chave atualmente em uso, com sua funcionalidade descrita.

- **`as`**: Realiza conversão primitiva, desambigua a trait específica que contém um item ou renomeia itens em instruções `use`.
- **`async`**: Retorna um `Future` em vez de bloquear a thread atual.
- **`await`**: Suspende a execução até que o resultado de um `Future` esteja pronto.
- **`break`**: Sai de um loop imediatamente.
- **`const`**: Define itens constantes ou ponteiros brutos constantes.
- **`continue`**: Continua para a próxima iteração do loop.
- **`crate`**: Em um caminho de módulo, refere-se à raiz do crate.
- **`dyn`**: Despacho dinâmico para um trait object.
- **`else`**: Alternativa para construções de fluxo de controle `if` e `if let`.
- **`enum`**: Define uma enumeração.
- **`extern`**: Vincula uma função ou variável externa.
- **`false`**: Literal booleano falso.
- **`fn`**: Define uma função ou o tipo de ponteiro de função.
- **`for`**: Itera sobre itens de um iterator, implementa uma trait ou especifica um lifetime de ordem superior.
- **`if`**: Ramifica com base no resultado de uma expressão condicional.
- **`impl`**: Implementa funcionalidade inerente ou de trait.
- **`in`**: Parte da sintaxe de loop `for`.
- **`let`**: Vincula uma variável.
- **`loop`**: Executa um loop incondicionalmente.
- **`match`**: Casa um valor com padrões.
- **`mod`**: Define um módulo.
- **`move`**: Faz um closure tomar ownership de todas as suas capturas.
- **`mut`**: Indica mutabilidade em referências, ponteiros brutos ou bindings de padrão.
- **`pub`**: Indica visibilidade pública em campos de struct, blocos `impl` ou módulos.
- **`ref`**: Vincula por referência.
- **`return`**: Retorna de uma função.
- **`Self`**: Um alias de tipo para o tipo que estamos definindo ou implementando.
- **`self`**: Sujeito de método ou módulo atual.
- **`static`**: Variável global ou lifetime que dura toda a execução do programa.
- **`struct`**: Define uma estrutura.
- **`super`**: Módulo pai do módulo atual.
- **`trait`**: Define uma trait.
- **`true`**: Literal booleano verdadeiro.
- **`type`**: Define um alias de tipo ou tipo associado.
- **`union`**: Define uma [union](https://doc.rust-lang.org/reference/items/unions.html); é palavra-chave apenas quando usada em uma declaração de union.
- **`unsafe`**: Indica código, funções, traits ou implementações _unsafe_.
- **`use`**: Traz símbolos para o escopo.
- **`where`**: Indica cláusulas que restringem um tipo.
- **`while`**: Executa um loop condicionalmente com base no resultado de uma expressão.

### Palavras-chave reservadas para uso futuro

As palavras-chave a seguir ainda não têm funcionalidade, mas estão reservadas pelo Rust para uso potencial futuro:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Identificadores brutos

_Identificadores brutos_ são a sintaxe que permite usar palavras-chave onde normalmente não seriam permitidas. Você usa um identificador bruto prefixando uma palavra-chave com `r#`.

Por exemplo, `match` é uma palavra-chave. Se você tentar compilar a seguinte função que usa `match` como nome:

**Arquivo: src/main.rs**

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

você receberá este erro:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

O erro mostra que você não pode usar a palavra-chave `match` como identificador da função. Para usar `match` como nome de função, você precisa usar a sintaxe de identificador bruto, assim:

**Arquivo: src/main.rs**

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Este código compilará sem erros. Observe o prefixo `r#` no nome da função em sua definição, bem como onde a função é chamada em `main`.

Identificadores brutos permitem usar qualquer palavra que você escolher como identificador, mesmo que essa palavra seja uma palavra-chave reservada. Isso nos dá mais liberdade para escolher nomes de identificadores e também nos permite integrar com programas escritos em uma linguagem em que essas palavras não são palavras-chave. Além disso, identificadores brutos permitem usar bibliotecas escritas em uma edição diferente do Rust da usada pelo seu crate. Por exemplo, `try` não é palavra-chave na edição 2015, mas é nas edições 2018, 2021 e 2024. Se você depender de uma biblioteca escrita usando a edição 2015 e ela tiver uma função `try`, precisará usar a sintaxe de identificador bruto, `r#try` neste caso, para chamar essa função a partir do seu código em edições posteriores. Consulte o [Apêndice E](/livro/cap22-05-edicoes) para mais informações sobre edições.
