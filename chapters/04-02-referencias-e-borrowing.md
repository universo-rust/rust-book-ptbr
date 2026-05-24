---
title: "Referências e Borrowing"
chapter_code: 04-02
slug: referencias-e-borrowing
---

# Referências e Borrowing

O problema com o código de tupla na Listagem 4-5 é que precisamos devolver a `String` para a função chamadora para que ainda possamos usar a `String` depois da chamada a `calculate_length`, porque a `String` foi movida para dentro de `calculate_length`. Em vez disso, podemos fornecer uma referência ao valor da `String`. Uma referência é como um ponteiro, no sentido de que é um endereço que podemos seguir para acessar os dados armazenados nesse endereço; esses dados são de propriedade de alguma outra variável. Diferentemente de um ponteiro, uma referência tem a garantia de apontar para um valor válido de um tipo particular durante toda a vida dessa referência.

Veja como você definiria e usaria uma função `calculate_length` que recebe uma referência a um objeto como parâmetro, em vez de tomar ownership do valor:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{s1}' is {len}.");
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Primeiro, observe que todo o código de tupla na declaração da variável e no valor de retorno da função desapareceu. Em segundo lugar, note que passamos `&s1` para `calculate_length` e, em sua definição, recebemos `&String` em vez de `String`. Esses e-comerciais (`&`) representam referências e permitem que você se refira a algum valor sem tomar ownership dele. A Figura 4-6 ilustra esse conceito.

![Três tabelas: a tabela de s contém apenas um ponteiro para a tabela de s1. A tabela de s1 contém os dados na stack de s1 e aponta para os dados da string na heap.](https://doc.rust-lang.org/book/img/trpl04-06.svg)

*Figura 4-6: Diagrama de `&String` `s` apontando para `String` `s1`*

> **Nota:** O oposto de referenciar usando `&` é _dereferenciar_, o que é feito com o operador de dereferência, `*`. Veremos alguns usos do operador de dereferência no Capítulo 8 e discutiremos os detalhes de dereferenciação no Capítulo 15.

Vamos dar uma olhada mais de perto na chamada de função aqui:

```rust
let s1 = String::from("hello");

let len = calculate_length(&s1);
```

A sintaxe `&s1` nos permite criar uma referência que _se refere_ ao valor de `s1`, mas não é dona dele. Como a referência não é dona do valor, o valor para o qual ela aponta não será descartado quando a referência deixar de ser usada.

Da mesma forma, a assinatura da função usa `&` para indicar que o tipo do parâmetro `s` é uma referência. Vamos adicionar algumas anotações explicativas:

```rust
fn calculate_length(s: &String) -> usize { // s é uma referência a uma String
    s.len()
} // Aqui, s sai de escopo. Mas como não tem ownership do que
  // se refere, ela não é descartada.
```

O escopo no qual a variável `s` é válida é o mesmo de qualquer parâmetro de função, mas o valor apontado pela referência não é descartado quando `s` deixa de ser usada, porque `s` não tem ownership. Quando funções têm referências como parâmetros em vez dos valores reais, não precisaremos devolver os valores para devolver o ownership, porque nunca tivemos ownership.

Chamamos a ação de criar uma referência de _borrowing_. Como na vida real, se uma pessoa possui algo, você pode pedir emprestado. Quando terminar, você precisa devolver. Você não é o dono.

Então, o que acontece se tentarmos modificar algo que estamos emprestando? Experimente o código na Listagem 4-6. Spoiler: não funciona!

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

<a id="listagem-4-6"></a>

[Listagem 4-6](#listagem-4-6): Tentando modificar um valor emprestado

Veja o erro:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable
  |
help: consider changing this to be a mutable reference
  |
7 | fn change(some_string: &mut String) {
  |                         +++

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Assim como as variáveis são imutáveis por padrão, as referências também são. Não temos permissão para modificar algo para o qual temos uma referência.

## Referências mutáveis

Podemos corrigir o código da Listagem 4-6 para nos permitir modificar um valor emprestado com apenas alguns pequenos ajustes que usam, em vez disso, uma _referência mutável_:

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Primeiro, mudamos `s` para `mut`. Depois, criamos uma referência mutável com `&mut s` onde chamamos a função `change` e atualizamos a assinatura da função para aceitar uma referência mutável com `some_string: &mut String`. Isso deixa muito claro que a função `change` vai mutar o valor que ela empresta.

Referências mutáveis têm uma grande restrição: se você tem uma referência mutável a um valor, não pode ter nenhuma outra referência a esse valor. Este código que tenta criar duas referências mutáveis a `s` vai falhar:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

Veja o erro:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Este erro diz que este código é inválido porque não podemos emprestar `s` como mutável mais de uma vez ao mesmo tempo. O primeiro empréstimo mutável está em `r1` e deve durar até ser usado no `println!`, mas entre a criação dessa referência mutável e seu uso, tentamos criar outra referência mutável em `r2` que empresta os mesmos dados que `r1`.

A restrição que impede múltiplas referências mutáveis aos mesmos dados ao mesmo tempo permite mutação, mas de forma muito controlada. É algo com o qual novos Rustaceans têm dificuldade, porque a maioria das linguagens permite mutar quando quiser. O benefício de ter essa restrição é que o Rust pode prevenir _data races_ em tempo de compilação. Uma _data race_ é similar a uma condição de corrida (_race condition_) e acontece quando estes três comportamentos ocorrem:

- Dois ou mais ponteiros acessam os mesmos dados ao mesmo tempo.
- Pelo menos um dos ponteiros está sendo usado para escrever nos dados.
- Não há nenhum mecanismo sendo usado para sincronizar o acesso aos dados.

Data races causam comportamento indefinido e podem ser difíceis de diagnosticar e corrigir quando você tenta rastreá-las em tempo de execução; o Rust previne esse problema recusando-se a compilar código com data races!

Como sempre, podemos usar chaves para criar um novo escopo, permitindo múltiplas referências mutáveis, só que não _simultâneas_:

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
} // r1 sai de escopo aqui, então podemos criar uma nova referência sem problemas.

let r2 = &mut s;
```

O Rust impõe uma regra similar para combinar referências mutáveis e imutáveis. Este código resulta em um erro:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let mut s = String::from("hello");

let r1 = &s; // sem problemas
let r2 = &s; // sem problemas
let r3 = &mut s; // GRANDE PROBLEMA

println!("{}, {}, and {}", r1, r2, r3);
```

Veja o erro:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s;
  |              -- immutable borrow occurs here
5 |     let r2 = &s;
6 |     let r3 = &mut s;
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Ufa! _Também_ não podemos ter uma referência mutável enquanto temos uma referência imutável para o mesmo valor.

Usuários de uma referência imutável não esperam que o valor mude de repente por baixo deles! No entanto, múltiplas referências imutáveis são permitidas porque ninguém que está apenas lendo os dados tem a capacidade de afetar a leitura de outra pessoa.

Note que o escopo de uma referência começa de onde ela é introduzida e continua até a última vez que essa referência é usada. Por exemplo, este código vai compilar porque o último uso das referências imutáveis está no `println!`, antes de a referência mutável ser introduzida:

```rust
let mut s = String::from("hello");

let r1 = &s; // sem problemas
let r2 = &s; // sem problemas
println!("{} and {}", r1, r2);
// r1 e r2 não são mais usadas depois deste ponto

let r3 = &mut s; // sem problemas
println!("{}", r3);
```

Os escopos das referências imutáveis `r1` e `r2` terminam depois do `println!` onde elas são usadas pela última vez, o que é antes de a referência mutável `r3` ser criada. Esses escopos não se sobrepõem, então este código é permitido: o compilador consegue dizer que a referência não está mais sendo usada em um ponto antes do fim do escopo.

Embora erros de borrowing possam ser frustrantes às vezes, lembre-se de que é o compilador Rust apontando um bug potencial cedo (em tempo de compilação, em vez de em tempo de execução) e mostrando exatamente onde está o problema. Assim, você não precisa descobrir por que seus dados não são o que você pensava que eram.

## Referências dangling

Em linguagens com ponteiros, é fácil criar erroneamente um _ponteiro dangling_ — um ponteiro que referencia uma localização na memória que pode ter sido entregue a outra pessoa — ao liberar alguma memória enquanto preserva um ponteiro para essa memória. No Rust, por outro lado, o compilador garante que referências nunca serão referências dangling: se você tem uma referência a alguns dados, o compilador vai garantir que os dados não sairão de escopo antes da referência a esses dados.

Vamos tentar criar uma referência dangling para ver como o Rust as previne com um erro em tempo de compilação:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Veja o erro:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime, but this is uncommon unless referencing static data
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Esta mensagem de erro se refere a um recurso que ainda não cobrimos: lifetimes. Discutiremos lifetimes em detalhes no Capítulo 10. Mas, se você ignorar as partes sobre lifetimes, a mensagem contém a chave para entender por que este código é um problema:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Vamos dar uma olhada mais de perto em exatamente o que está acontecendo em cada etapa do nosso código `dangle`:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn dangle() -> &String { // dangle retorna uma referência a uma String

    let s = String::from("hello"); // s é uma nova String

    &s // retornamos uma referência à String s
} // Aqui, s sai de escopo e é descartada. Sua memória desaparece.
  // Perigo!
```

Como `s` é criada dentro de `dangle`, quando o código de `dangle` termina, `s` será desalocada. Mas tentamos retornar uma referência a ela. Isso significa que esta referência estaria apontando para uma `String` inválida. Isso não é bom! O Rust não nos deixa fazer isso.

A solução aqui é retornar a `String` diretamente:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

Isso funciona sem nenhum problema. O ownership é movido para fora, e nada é desalocado.

## As regras das referências

Vamos recapitular o que discutimos sobre referências:

- Em qualquer momento, você pode ter _ou_ uma referência mutável _ou_ qualquer número de referências imutáveis.
- Referências devem ser sempre válidas.

A seguir, veremos um tipo diferente de referência: slices.
