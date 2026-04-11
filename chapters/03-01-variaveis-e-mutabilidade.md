---
title: "Variáveis e mutabilidade"
chapter_code: 03-01
slug: variaveis-e-mutabilidade
---

# Variáveis e mutabilidade

Como mencionado na seção *"Armazenando valores com variáveis"*, por padrão, as variáveis em Rust são imutáveis. Esse é um dos vários "empurrõezinhos" que o Rust dá para incentivar você a escrever código que aproveite melhor a segurança e a facilidade de concorrência que a linguagem oferece. Ainda assim, você tem a opção de tornar suas variáveis mutáveis. Vamos explorar como e por que o Rust incentiva o uso da imutabilidade e por que, às vezes, você pode querer abrir mão disso.

Quando uma variável é imutável, depois que um valor é associado a um nome, esse valor não pode ser alterado. Para ilustrar isso, crie um novo projeto chamado `variables` dentro do diretório de projetos usando o comando: `cargo new variables`

Em seguida, dentro do diretório `variables`, abra o arquivo `src/main.rs` e substitua o código pelo seguinte (que ainda **não vai compilar**):

Arquivo: src/main.rs
```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```
[Listagem 1](#): Exemplo de código tentando alterar uma variável imutável

Salve o arquivo e execute o programa com:

```bash
$ cargo run
```

Você deverá receber uma mensagem de erro relacionada à imutabilidade, parecida com esta:

```
error[E0384]: cannot assign twice to immutable variable `x`
```

Esse exemplo mostra como o compilador ajuda você a encontrar erros no seu programa. Erros de compilação podem ser frustrantes, mas eles só significam que o programa ainda não está fazendo, de forma segura, o que você quer que ele faça — isso não quer dizer que você seja um mau programador. Até programadores experientes em Rust ainda recebem erros do compilador.

Você recebeu a mensagem de erro *cannot assign twice to immutable variable `x`* porque tentou atribuir um segundo valor à variável `x`, que é imutável.

> **Nota:** Erros detectados em tempo de compilação evitam bugs difíceis de encontrar mais tarde, especialmente quando diferentes partes do código fazem suposições diferentes sobre um mesmo valor.

É importante que esse tipo de erro seja detectado em tempo de compilação, porque essa situação pode facilmente levar a bugs. Imagine que uma parte do código assume que um valor nunca vai mudar, enquanto outra parte altera esse valor. O compilador do Rust garante que, quando você diz que um valor não vai mudar, ele realmente não muda, o que torna o código mais fácil de entender e de manter.

Por outro lado, a mutabilidade pode ser muito útil e tornar o código mais conveniente de escrever. Embora as variáveis sejam imutáveis por padrão, você pode torná-las mutáveis adicionando a palavra-chave `mut` antes do nome da variável, como você fez no [Capítulo 2](https://universorust.com.br/area-membro/livro-rust/2-programando-um-jogo-de-adivinhacao). Isso também deixa clara a sua intenção para quem for ler o código no futuro.

Veja o exemplo abaixo:

Arquivo: src/main.rs
```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```
[Listagem 2](#): Exemplo usando uma variável mutável

Ao executar o programa, o resultado será:

```
The value of x is: 5
The value of x is: 6
```

Nesse caso, é permitido alterar o valor associado a `x` de 5 para 6 porque usamos `mut`. No fim das contas, decidir se deve usar mutabilidade ou não depende de você e do que considera mais claro em cada situação.

## Declarando constantes

Assim como as variáveis imutáveis, constantes são valores associados a um nome e não podem ser alterados. Porém, existem algumas diferenças importantes entre constantes e variáveis.

Primeiro, não é permitido usar `mut` com constantes. As constantes não são apenas imutáveis por padrão — elas são sempre imutáveis. Para declarar uma constante, usamos a palavra-chave `const` em vez de `let`, e o tipo do valor deve ser informado obrigatoriamente. Vamos falar mais sobre tipos e anotações de tipo na próxima seção, [Tipos de dados](https://universorust.com.br/area-membro/livro-rust/3-2-tipos-de-dados), então não se preocupe com os detalhes agora. Por enquanto, basta saber que o tipo sempre precisa ser declarado.

Constantes podem ser declaradas em qualquer escopo, inclusive no escopo global. Isso as torna muito úteis para valores que precisam ser conhecidos por várias partes do código.

Outra diferença importante é que constantes só podem receber expressões constantes, ou seja, valores que podem ser calculados em tempo de compilação, e não resultados que só seriam conhecidos durante a execução do programa.

Veja um exemplo de declaração de constante:

Exemplo de constante em Rust
```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```
[Listagem 1](#): Declaração de uma constante com expressão avaliada em tempo de compilação

O nome da constante é `THREE_HOURS_IN_SECONDS`, e seu valor é definido multiplicando 60 (segundos em um minuto) por 60 (minutos em uma hora) por 3 (o número de horas que queremos representar).

A convenção de nomes do Rust para constantes é usar letras maiúsculas, com underscores separando as palavras. O compilador do Rust consegue avaliar um conjunto limitado de operações em tempo de compilação, o que permite escrever esse valor de forma mais clara e fácil de entender, em vez de simplesmente usar o número `10_800`.

Isso torna o código mais legível e menos propenso a erros. Para saber exatamente quais operações podem ser usadas, consulte a [seção sobre avaliação de constantes](https://doc.rust-lang.org/reference/const_eval.html) na documentação oficial do Rust.

As constantes são válidas durante todo o tempo de execução do programa, dentro do escopo em que foram declaradas. Essa característica faz com que elas sejam ideais para representar valores importantes do domínio da aplicação, que várias partes do programa precisam conhecer.

> **Nota:** Constantes são especialmente úteis para valores como limites máximos, configurações globais ou números que não devem mudar ao longo da execução do programa.

Dar nomes a valores fixos usados ao longo do programa, transformando-os em constantes, ajuda a deixar o significado desses valores mais claro para quem for manter o código no futuro. Além disso, se esse valor precisar ser alterado algum dia, você terá apenas um único lugar no código para fazer essa mudança.

## Sombreamento (shadowing)

Como você viu no tutorial do jogo de adivinhação no Capítulo 2, é possível declarar uma nova variável com o **mesmo nome** de uma variável anterior. Em Rust, dizemos que a primeira variável foi **sombreada** (*shadowed*) pela segunda. Isso significa que, a partir desse ponto, é a **variável mais recente** que o compilador considera quando você usa aquele nome.

Na prática, a segunda variável "cobre" a primeira, e todas as referências ao nome passam a se referir a ela, até que essa variável também seja sombreada ou até que o escopo termine. Podemos fazer isso reutilizando o mesmo nome e usando novamente a palavra-chave `let`, como no exemplo a seguir:

Arquivo: src/main.rs
```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```
[Listagem 1](#): Exemplo de sombreamento de variáveis em diferentes escopos

Nesse programa, primeiro associamos o valor `5` à variável `x`. Em seguida, criamos uma **nova variável `x`** usando novamente `let x =`, pegando o valor anterior e somando 1, o que faz com que `x` passe a valer `6`.

Depois disso, dentro de um escopo interno criado pelas chaves `{ }`, a terceira instrução `let` também sombreia `x`, criando mais uma nova variável. Dessa vez, o valor anterior é multiplicado por 2, fazendo com que `x` tenha o valor `12`. Quando esse escopo termina, o sombreamento acaba, e `x` volta a ter o valor `6`.

Ao executar o programa, a saída será:

```bash
$ cargo run
The value of x in the inner scope is: 12
The value of x is: 6
```

O sombreamento é diferente de tornar uma variável mutável com `mut`, pois o compilador gera um erro em tempo de compilação se você tentar reatribuir um valor sem usar a palavra-chave `let`. Com o sombreamento, podemos aplicar transformações a um valor e ainda manter a variável **imutável** depois disso.

> **Nota:** O sombreamento cria uma nova variável, enquanto `mut` permite alterar o valor da mesma variável.

Outra diferença importante entre `mut` e o sombreamento é que, ao criar uma nova variável com `let`, podemos **mudar o tipo do valor** e ainda reutilizar o mesmo nome.

Por exemplo, imagine um programa que receba espaços digitados pelo usuário e, depois, queira armazenar a quantidade desses espaços como um número:

Exemplo de mudança de tipo com sombreamento
```rust
let spaces = "   ";
let spaces = spaces.len();
```
[Listagem 2](#): Uso de sombreamento para mudar o tipo da variável

A primeira variável `spaces` é do tipo texto (`&str`), enquanto a segunda é um número (`usize`). Isso evita a necessidade de nomes diferentes, como `spaces_str` e `spaces_num`.

Se tentarmos fazer o mesmo usando `mut`, o código não irá compilar:

Código inválido usando mut
```rust
let mut spaces = "   ";
spaces = spaces.len();
```
[Listagem 3](#): Erro ao tentar alterar o tipo de uma variável mutável

O erro ocorre porque **não é permitido alterar o tipo de uma variável**.

```bash
$ cargo run
error[E0308]: mismatched types
```

Agora que já exploramos como as variáveis funcionam, podemos avançar para os outros tipos de dados que elas podem assumir.