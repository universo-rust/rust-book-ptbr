---
title: "Tipos avançados"
chapter_code: 20-03
slug: tipos-avancados
---

# Tipos Avançados

O sistema de tipos de Rust tem recursos que mencionamos, mas ainda não discutimos. Começaremos com newtypes em geral e por que são úteis como tipos. Depois veremos aliases de tipo, parecidos com newtypes mas com semântica ligeiramente diferente. Também discutiremos o tipo `!` e tipos de tamanho dinâmico.

## Segurança de tipos e abstração com o padrão newtype

Esta seção assume que você leu a seção anterior Implementando traits externas com o padrão newtype. O padrão newtype também serve para tarefas além das que vimos, incluindo garantir estaticamente que valores nunca se confundam e indicar as unidades de um valor. Você viu newtypes para unidades na Listagem 20-16: lembre-se de que as structs `Millimeters` e `Meters` envolviam valores `u32` em um newtype. Se escrevêssemos uma função com parâmetro `Millimeters`, não compilaríamos um programa que tentasse chamá-la com `Meters` ou um `u32` simples.

Também podemos usar o padrão newtype para abstrair detalhes de implementação de um tipo: o novo tipo pode expor uma API pública diferente da API do tipo interno privado.

Newtypes também podem ocultar implementação interna. Por exemplo, poderíamos fornecer um tipo `People` envolvendo `HashMap<i32, String>` que associa ID de pessoa ao nome. Código que usa `People` só interagiria com a API pública que fornecemos, como um método para adicionar um nome à coleção `People`; esse código não precisaria saber que atribuímos internamente um ID `i32` aos nomes. O padrão newtype é uma forma leve de encapsulamento para ocultar detalhes de implementação, discutido em Encapsulamento que esconde detalhes de implementação do Capítulo 18.

## Sinônimos de tipo e aliases de tipo

Rust permite declarar um _alias de tipo_ para dar a um tipo existente outro nome. Para isso usamos a palavra-chave `type`. Por exemplo, podemos criar o alias `Kilometers` para `i32` assim:

**Arquivo: src/main.rs**

```rust
    type Kilometers = i32;
```

Agora o alias `Kilometers` é um _sinônimo_ de `i32`; diferente dos tipos `Millimeters` e `Meters` da Listagem 20-16, `Kilometers` não é um tipo novo separado. Valores do tipo `Kilometers` são tratados igual a valores de `i32`:

**Arquivo: src/main.rs**

```rust
    let x: i32 = 5;
    let y: Kilometers = 5;

    println!("x + y = {}", x + y);
```

Como `Kilometers` e `i32` são o mesmo tipo, podemos somar valores dos dois tipos e passar valores `Kilometers` a funções que recebem parâmetros `i32`. Porém, com este método não ganhamos os benefícios de verificação de tipos do padrão newtype discutido antes. Ou seja, se confundirmos valores `Kilometers` e `i32` em algum lugar, o compilador não nos dará erro.

O principal caso de uso de sinônimos de tipo é reduzir repetição. Por exemplo, podemos ter um tipo longo como:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Escrever esse tipo longo em assinaturas de função e anotações de tipo por todo o código cansa e favorece erros. Imagine um projeto cheio de código como na Listagem 20-25.

**Arquivo: src/main.rs**

```rust
fn main() {
    let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

    fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
        // --snip--
    }

    fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
        // --snip--
    }
}
```

<a id="listagem-20-25"></a>

[Listagem 20-25](#listagem-20-25): Usando um tipo longo em muitos lugares

Um alias de tipo torna o código mais gerenciável ao reduzir repetição. Na Listagem 20-26 introduzimos um alias `Thunk` para o tipo verboso e substituímos todos os usos pelo alias mais curto `Thunk`.

**Arquivo: src/main.rs**

```rust
fn main() {
    type Thunk = Box<dyn Fn() + Send + 'static>;

    let f: Thunk = Box::new(|| println!("hi"));

    fn takes_long_type(f: Thunk) {
        // --snip--
    }

    fn returns_long_type() -> Thunk {
        // --snip--
    }
}
```

<a id="listagem-20-26"></a>

[Listagem 20-26](#listagem-20-26): Introduzindo o alias de tipo `Thunk` para reduzir repetição

Este código é bem mais fácil de ler e escrever! Escolher um nome significativo para um alias de tipo ajuda a comunicar a intenção (_thunk_ é um termo para código a ser avaliado depois, nome adequado para um closure armazenado).

Aliases de tipo também são comuns com `Result<T, E>` para reduzir repetição. Considere o módulo `std::io` da biblioteca padrão. Operações de I/O frequentemente retornam `Result<T, E>` para quando falham. A biblioteca tem a struct `std::io::Error` representando todos os erros de I/O possíveis. Muitas funções em `std::io` retornam `Result<T, E>` com `E` sendo `std::io::Error`, como estas funções da trait `Write`:

**Arquivo: src/lib.rs**

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

`Result<..., Error>` se repete muito. Por isso `std::io` tem esta declaração de alias de tipo:

**Arquivo: src/lib.rs**

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

Como esta declaração está no módulo `std::io`, podemos usar o alias totalmente qualificado `std::io::Result<T>` — ou seja, um `Result<T, E>` com `E` preenchido como `std::io::Error`. As assinaturas das funções da trait `Write` ficam assim:

**Arquivo: src/lib.rs**

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

O alias de tipo ajuda de duas formas: facilita escrever código _e_ dá uma interface consistente em todo `std::io`. Por ser um alias, continua sendo um `Result<T, E>`, então podemos usar métodos de `Result<T, E>` e sintaxe especial como o operador `?`.

## O tipo never que nunca retorna

Rust tem um tipo especial chamado `!`, conhecido em teoria de tipos como _tipo vazio_ porque não tem valores. Preferimos _tipo never_ porque ocupa o lugar do tipo de retorno quando uma função nunca retorna. Exemplo:

**Arquivo: src/lib.rs**

```rust
fn bar() -> ! {
    // --snip--
}
```

Este código se lê como “a função `bar` retorna never”. Funções que retornam never são _funções divergentes_. Não podemos criar valores do tipo `!`, então `bar` nunca pode retornar.

Mas qual a utilidade de um tipo para o qual nunca criamos valores? Lembre-se do código da Listagem 2-5, parte do jogo de adivinhação; reproduzimos um trecho na Listagem 20-27.

**Arquivo: src/main.rs**

```rust
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // --snip--
```

<a id="listagem-20-27"></a>

[Listagem 20-27](#listagem-20-27): Um `match` com braço que termina em `continue`

Na época, pulamos alguns detalhes. Em A construção de fluxo de controle `match` do Capítulo 6, vimos que braços de `match` devem retornar o mesmo tipo. Por exemplo, o código abaixo não funciona:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
    let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => "hello",
    };
```

O tipo de `guess` neste código teria de ser inteiro _e_ string, e Rust exige que `guess` tenha um só tipo. Então, o que `continue` retorna? Como pudemos retornar `u32` de um braço e ter outro que termina com `continue` na Listagem 20-27?

Como você pode imaginar, `continue` tem valor `!`. Ou seja, quando Rust calcula o tipo de `guess`, olha os dois braços do `match`: o primeiro com valor `u32` e o segundo com valor `!`. Como `!` nunca pode ter valor, Rust decide que o tipo de `guess` é `u32`.

A forma formal de descrever isso é que expressões do tipo `!` podem ser coercidas para qualquer outro tipo. Podemos terminar este braço de `match` com `continue` porque `continue` não retorna valor; devolve o controle ao topo do loop, então no caso `Err` nunca atribuímos valor a `guess`.

O tipo never também é útil com a macro `panic!`. Lembre-se de `unwrap` em valores `Option<T>`, que produz um valor ou entra em pânico, com esta definição:

**Arquivo: src/lib.rs**

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

Neste código, o mesmo ocorre que no `match` da Listagem 20-27: Rust vê que `val` tem tipo `T` e `panic!` tem tipo `!`, então o resultado da expressão `match` inteira é `T`. Funciona porque `panic!` não produz valor; encerra o programa. No caso `None`, não retornamos valor de `unwrap`, então o código é válido.

Uma última expressão com tipo `!` é um loop:

**Arquivo: src/main.rs**

```rust
    print!("forever ");

    loop {
        print!("and ever ");
    }
```

Aqui o loop nunca termina, então `!` é o valor da expressão. Porém, isso não seria verdade se incluíssemos um `break`, porque o loop terminado terminaria ao chegar no `break`.

## Tipos de tamanho dinâmico e a trait `Sized`

Rust precisa saber certos detalhes sobre seus tipos, como quanto espaço alocar para um valor de um tipo particular. Isso deixa um canto do sistema de tipos um pouco confuso no início: o conceito de _tipos de tamanho dinâmico_. Às vezes chamados de _DSTs_ ou _tipos unsized_, permitem escrever código com valores cujo tamanho só sabemos em runtime.

Vamos aos detalhes de um tipo de tamanho dinâmico chamado `str`, que usamos ao longo do livro. Isso mesmo: não `&str`, mas `str` sozinho, é um DST. Em muitos casos, como ao armazenar texto digitado pelo usuário, não sabemos o comprimento da string até runtime. Isso significa que não podemos criar variável do tipo `str`, nem receber argumento do tipo `str`. Considere o código abaixo, que não funciona:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
    let s1: str = "Hello there!";
    let s2: str = "How's it going?";
```

Rust precisa saber quanta memória alocar para qualquer valor de um tipo particular, e todos os valores de um tipo devem usar a mesma quantidade de memória. Se Rust permitisse este código, esses dois valores `str` precisariam ocupar o mesmo espaço. Mas têm comprimentos diferentes: `s1` precisa de 12 bytes de armazenamento e `s2` de 15. Por isso não é possível criar variável guardando um tipo de tamanho dinâmico.

Então, o que fazemos? Neste caso, você já sabe a resposta: fazemos o tipo de `s1` e `s2` ser fatia de string (`&str`) em vez de `str`. Lembre-se de String slices do Capítulo 4: a estrutura de fatia guarda só a posição inicial e o comprimento. Embora `&T` seja um valor único com o endereço de memória onde `T` está, uma fatia de string são _dois_ valores: o endereço do `str` e seu comprimento. Assim, sabemos o tamanho de uma fatia de string em compile time: é o dobro do comprimento de um `usize`. Ou seja, sempre sabemos o tamanho de uma fatia de string, não importa o comprimento da string referenciada. Em geral, é assim que tipos de tamanho dinâmico são usados em Rust: têm metadados extras que guardam o tamanho da informação dinâmica. A regra de ouro dos tipos de tamanho dinâmico é que devemos sempre colocar valores de tipos de tamanho dinâmico atrás de algum tipo de ponteiro.

Podemos combinar `str` com vários tipos de ponteiros: por exemplo, `Box<str>` ou `Rc<str>`. Na verdade, você já viu isso, mas com outro tipo de tamanho dinâmico: traits. Toda trait é um tipo de tamanho dinâmico a que podemos nos referir pelo nome da trait. Em Usando trait objects para abstrair comportamento compartilhado do Capítulo 18, mencionamos que para usar traits como trait objects, devemos colocá-las atrás de um ponteiro, como `&dyn Trait` ou `Box<dyn Trait>` (`Rc<dyn Trait>` também funcionaria).

Para trabalhar com DSTs, Rust fornece a trait `Sized` para determinar se o tamanho de um tipo é conhecido em compile time. Esta trait é implementada automaticamente para tudo cujo tamanho é conhecido em compile time. Além disso, Rust adiciona implicitamente um bound em `Sized` a toda função genérica. Ou seja, uma definição de função genérica como esta:

**Arquivo: src/lib.rs**

```rust
fn generic<T>(t: T) {
    // --snip--
}
```

é tratada como se tivéssemos escrito isto:

**Arquivo: src/lib.rs**

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

Por padrão, funções genéricas só funcionam em tipos com tamanho conhecido em compile time. Porém, você pode usar a sintaxe especial abaixo para relaxar essa restrição:

**Arquivo: src/lib.rs**

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

Um trait bound em `?Sized` significa “`T` pode ou não ser `Sized`”, e essa notação substitui o padrão de que tipos genéricos devem ter tamanho conhecido em compile time. A sintaxe `?Trait` com este significado só está disponível para `Sized`, não para outras traits.

Note também que mudamos o tipo do parâmetro `t` de `T` para `&T`. Como o tipo pode não ser `Sized`, precisamos usá-lo atrás de algum ponteiro. Neste caso, escolhemos uma referência.

A seguir, falaremos sobre funções e closures!
