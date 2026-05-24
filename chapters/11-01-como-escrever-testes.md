---
title: "Como escrever testes"
chapter_code: 11-01
slug: como-escrever-testes
---

# Como Escrever Testes

_Testes_ são funções Rust que verificam se o código que não é teste está funcionando da maneira esperada. Os corpos das funções de teste normalmente executam estas três ações:

- Configurar quaisquer dados ou estado necessários.
- Executar o código que você quer testar.
- Afirmar que os resultados são o que você espera.

Vamos ver os recursos que o Rust fornece especificamente para escrever testes que realizam essas ações, incluindo o atributo `test`, algumas macros e o atributo `should_panic`.

### Estruturando funções de teste

No mais simples, um teste em Rust é uma função anotada com o atributo `test`. Atributos são metadados sobre pedaços de código Rust; um exemplo é o atributo `derive` que usamos com structs no Capítulo 5. Para transformar uma função em função de teste, adicione `#[test]` na linha antes de `fn`. Quando você executa seus testes com o comando `cargo test`, o Rust compila um binário test runner que executa as funções anotadas e informa se cada função de teste passou ou falhou.

Sempre que criamos um novo projeto de biblioteca com Cargo, um módulo de teste com uma função de teste é gerado automaticamente para nós. Este módulo oferece um modelo para escrever seus testes, para que você não precise procurar a estrutura e a sintaxe exatas toda vez que iniciar um novo projeto. Você pode adicionar quantas funções de teste e módulos de teste adicionais quiser!

Exploraremos alguns aspectos de como os testes funcionam experimentando com o teste modelo antes de testar código de verdade. Depois, escreveremos alguns testes do mundo real que chamam código que escrevemos e afirmam que seu comportamento está correto.

Vamos criar um novo projeto de biblioteca chamado `adder` que somará dois números:

```bash
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

O conteúdo do arquivo _src/lib.rs_ na sua biblioteca `adder` deve parecer com a Listagem 11-1.

**Arquivo: src/lib.rs**

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

<a id="listagem-11-1"></a>

[Listagem 11-1](#listagem-11-1): O código gerado automaticamente por `cargo new`

O arquivo começa com uma função de exemplo `add` para que tenhamos algo para testar.

Por enquanto, vamos focar apenas na função `it_works`. Observe a anotação `#[test]`: este atributo indica que esta é uma função de teste, então o test runner sabe que deve tratar esta função como teste. Também podemos ter funções que não são testes no módulo `tests` para ajudar a configurar cenários comuns ou realizar operações comuns, então sempre precisamos indicar quais funções são testes.

O corpo da função de exemplo usa a macro `assert_eq!` para afirmar que `result`, que contém o resultado de chamar `add` com 2 e 2, é igual a 4. Esta asserção serve como exemplo do formato de um teste típico. Vamos executá-lo para ver que este teste passa.

O comando `cargo test` executa todos os testes do nosso projeto, como mostrado na Listagem 11-2.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests src/lib.rs (target/debug/deps/adder-01ad14159ff659ab)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

<a id="listagem-11-2"></a>

[Listagem 11-2](#listagem-11-2): A saída de executar o teste gerado automaticamente

O Cargo compilou e executou o teste. Vemos a linha `running 1 test`. A linha seguinte mostra o nome da função de teste gerada, chamada `tests::it_works`, e que o resultado de executar esse teste é `ok`. O resumo geral `test result: ok.` significa que todos os testes passaram, e a parte que diz `1 passed; 0 failed` totaliza o número de testes que passaram ou falharam.

É possível marcar um teste como ignorado para que não execute em uma instância particular; cobriremos isso na seção Ignorando testes a menos que especificamente solicitado mais adiante neste capítulo. Como não fizemos isso aqui, o resumo mostra `0 ignored`. Também podemos passar um argumento ao comando `cargo test` para executar apenas testes cujo nome corresponda a uma string; isso é chamado de _filtragem_, e cobriremos na seção Executando um subconjunto de testes por nome. Aqui, não filtramos os testes executados, então o fim do resumo mostra `0 filtered out`.

A estatística `0 measured` é para testes de benchmark que medem desempenho. Testes de benchmark, no momento em que escrevemos, estão disponíveis apenas no Rust nightly. Consulte a [documentação sobre testes de benchmark](https://doc.rust-lang.org/nightly/unstable-book/library-features/test.html) para saber mais.

A próxima parte da saída de teste, começando em `Doc-tests adder`, é para os resultados de quaisquer testes de documentação. Ainda não temos testes de documentação, mas o Rust pode compilar quaisquer exemplos de código que apareçam na documentação da nossa API. Este recurso ajuda a manter sua documentação e seu código sincronizados! Discutiremos como escrever testes de documentação na seção Comentários de documentação como testes do Capítulo 14. Por enquanto, ignoraremos a saída `Doc-tests`.

Vamos começar a personalizar o teste às nossas necessidades. Primeiro, mude o nome da função `it_works` para um nome diferente, como `exploration`, assim:

**Arquivo: src/lib.rs**

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn exploration() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

Depois, execute `cargo test` novamente. A saída agora mostra `exploration` em vez de `it_works`:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.59s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Agora adicionaremos outro teste, mas desta vez faremos um teste que falha! Testes falham quando algo na função de teste entra em pânico. Cada teste é executado em uma nova thread, e quando a thread principal vê que uma thread de teste morreu, o teste é marcado como falha. No Capítulo 9, falamos que a forma mais simples de entrar em pânico é chamar a macro `panic!`. Insira o novo teste como função chamada `another`, para que seu arquivo _src/lib.rs_ fique como na Listagem 11-3.

**Arquivo: src/lib.rs (Este código entra em pânico!)**

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn exploration() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<a id="listagem-11-3"></a>

[Listagem 11-3](#listagem-11-3): Adicionando um segundo teste que falhará porque chamamos a macro `panic!`

Execute os testes novamente com `cargo test`. A saída deve parecer com a Listagem 11-4, que mostra que nosso teste `exploration` passou e `another` falhou.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----

thread 'tests::another' panicked at src/lib.rs:17:9:
Make this test fail
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

<a id="listagem-11-4"></a>

[Listagem 11-4](#listagem-11-4): Resultados de teste quando um teste passa e outro falha

Em vez de `ok`, a linha `test tests::another` mostra `FAILED`. Duas novas seções aparecem entre os resultados individuais e o resumo: a primeira exibe o motivo detalhado de cada falha de teste. Neste caso, obtemos os detalhes de que `tests::another` falhou porque entrou em pânico com a mensagem `Make this test fail` na linha 17 do arquivo _src/lib.rs_. A seção seguinte lista apenas os nomes de todos os testes que falharam, o que é útil quando há muitos testes e muita saída detalhada de falhas. Podemos usar o nome de um teste que falhou para executar apenas esse teste e depurá-lo mais facilmente; falaremos mais sobre formas de executar testes na seção Controlando como os testes são executados.

A linha de resumo aparece no final: no geral, nosso resultado de teste é `FAILED`. Tivemos um teste passando e um falhando.

Agora que você viu como os resultados de teste parecem em cenários diferentes, vamos ver algumas macros além de `panic!` que são úteis em testes.

### Verificando resultados com `assert!`

A macro `assert!`, fornecida pela biblioteca padrão, é útil quando você quer garantir que alguma condição em um teste avalie para `true`. Damos à macro `assert!` um argumento que avalia para um booleano. Se o valor for `true`, nada acontece e o teste passa. Se o valor for `false`, a macro `assert!` chama `panic!` para fazer o teste falhar. Usar a macro `assert!` nos ajuda a verificar que nosso código está funcionando da forma que pretendemos.

No Capítulo 5, na Listagem 5-15, usamos uma struct `Rectangle` e um método `can_hold`, que repetimos aqui na Listagem 11-5. Vamos colocar este código no arquivo _src/lib.rs_ e então escrever alguns testes para ele usando a macro `assert!`.

**Arquivo: src/lib.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<a id="listagem-11-5"></a>

[Listagem 11-5](#listagem-11-5): A struct `Rectangle` e seu método `can_hold` do Capítulo 5

O método `can_hold` retorna um booleano, o que significa que é um caso de uso perfeito para a macro `assert!`. Na Listagem 11-6, escrevemos um teste que exercita o método `can_hold` criando uma instância de `Rectangle` com largura 8 e altura 7 e afirmando que ela pode conter outra instância de `Rectangle` com largura 5 e altura 1.

**Arquivo: src/lib.rs**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```

<a id="listagem-11-6"></a>

[Listagem 11-6](#listagem-11-6): Um teste para `can_hold` que verifica se um retângulo maior pode de fato conter um menor

Observe a linha `use super::*;` dentro do módulo `tests`. O módulo `tests` é um módulo regular que segue as regras de visibilidade usuais que cobrimos no Capítulo 7 na seção Caminhos para referenciar um item na árvore de módulos. Como o módulo `tests` é um módulo interno, precisamos trazer o código sob teste no módulo externo para o escopo do módulo interno. Usamos um glob aqui, então qualquer coisa que definimos no módulo externo está disponível para este módulo `tests`.

Nomeamos nosso teste `larger_can_hold_smaller` e criamos as duas instâncias de `Rectangle` que precisamos. Depois, chamamos a macro `assert!` e passamos o resultado de chamar `larger.can_hold(&smaller)`. Esta expressão deve retornar `true`, então nosso teste deve passar. Vamos descobrir!

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Passou! Vamos adicionar outro teste, desta vez afirmando que um retângulo menor não pode conter um maior:

**Arquivo: src/lib.rs**

```rust
    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
```

Como o resultado correto da função `can_hold` neste caso é `false`, precisamos negar esse resultado antes de passá-lo à macro `assert!`. Assim, nosso teste passará se `can_hold` retornar `false`:

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... ok
test tests::smaller_cannot_hold_larger ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Dois testes que passam! Agora vejamos o que acontece com os resultados dos testes quando introduzimos um bug no código. Mudaremos a implementação do método `can_hold` substituindo o sinal de maior (`>`) por menor (`<`) ao comparar as larguras:

**Arquivo: src/lib.rs**

```rust
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}
```

Executar os testes agora produz o seguinte:

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... FAILED
test tests::smaller_cannot_hold_larger ... ok

failures:

---- tests::larger_can_hold_smaller stdout ----

thread 'tests::larger_can_hold_smaller' panicked at src/lib.rs:28:9:
assertion failed: larger.can_hold(&smaller)
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Nossos testes pegaram o bug! Como `larger.width` é `8` e `smaller.width` é `5`, a comparação das larguras em `can_hold` agora retorna `false`: 8 não é menor que 5.

### Testando igualdade com `assert_eq!` e `assert_ne!`

Uma forma comum de verificar funcionalidade é testar igualdade entre o resultado do código sob teste e o valor que você espera que o código retorne. Você poderia fazer isso usando a macro `assert!` e passando uma expressão com o operador `==`. Porém, este é um teste tão comum que a biblioteca padrão fornece um par de macros — `assert_eq!` e `assert_ne!` — para realizar este teste com mais conveniência. Essas macros comparam dois argumentos quanto à igualdade ou desigualdade, respectivamente. Elas também imprimirão os dois valores se a asserção falhar, o que facilita ver _por que_ o teste falhou; por outro lado, a macro `assert!` só indica que obteve valor `false` para a expressão `==`, sem imprimir os valores que levaram ao `false`.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao seu parâmetro e então testamos esta função usando a macro `assert_eq!`.

**Arquivo: src/lib.rs**

```rust
pub fn add_two(a: u64) -> u64 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        let result = add_two(2);
        assert_eq!(result, 4);
    }
}
```

<a id="listagem-11-7"></a>

[Listagem 11-7](#listagem-11-7): Testando a função `add_two` usando a macro `assert_eq!`

Vamos verificar se passa!

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Criamos uma variável chamada `result` que contém o resultado de chamar `add_two(2)`. Depois, passamos `result` e `4` como argumentos à macro `assert_eq!`. A linha de saída deste teste é `test tests::it_adds_two ... ok`, e o texto `ok` indica que nosso teste passou!

Vamos introduzir um bug no código para ver como `assert_eq!` parece quando falha. Mude a implementação da função `add_two` para adicionar `3` em vez disso:

**Arquivo: src/lib.rs**

```rust
pub fn add_two(a: u64) -> u64 {
    a + 3
}
```

Execute os testes novamente:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----

thread 'tests::it_adds_two' panicked at src/lib.rs:12:9:
assertion `left == right` failed
  left: 5
 right: 4
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Nosso teste pegou o bug! O teste `tests::it_adds_two` falhou, e a mensagem nos diz que a asserção que falhou foi `left == right` e quais são os valores de `left` e `right`. Esta mensagem nos ajuda a começar a depurar: o argumento `left`, onde tínhamos o resultado de chamar `add_two(2)`, foi `5`, mas o argumento `right` foi `4`. Você pode imaginar que isso seria especialmente útil quando temos muitos testes em execução.

Observe que em algumas linguagens e frameworks de teste, os parâmetros das funções de asserção de igualdade são chamados de `expected` e `actual`, e a ordem em que especificamos os argumentos importa. Porém, em Rust, eles são chamados de `left` e `right`, e a ordem em que especificamos o valor que esperamos e o valor que o código produz não importa. Poderíamos escrever a asserção neste teste como `assert_eq!(4, result)`, o que resultaria na mesma mensagem de falha que exibe `` assertion `left == right` failed ``.

A macro `assert_ne!` passará se os dois valores que damos não forem iguais e falhará se forem iguais. Esta macro é mais útil para casos em que não temos certeza de qual será um valor, mas sabemos qual valor definitivamente _não_ deveria ser. Por exemplo, se estamos testando uma função que garantidamente altera sua entrada de alguma forma, mas a forma como a entrada é alterada depende do dia da semana em que executamos os testes, a melhor coisa a afirmar pode ser que a saída da função não é igual à entrada.

Por baixo dos panos, as macros `assert_eq!` e `assert_ne!` usam os operadores `==` e `!=`, respectivamente. Quando as asserções falham, essas macros imprimem seus argumentos usando formatação de debug, o que significa que os valores comparados devem implementar as traits `PartialEq` e `Debug`. Todos os tipos primitivos e a maioria dos tipos da biblioteca padrão implementam essas traits. Para structs e enums que você define, precisará implementar `PartialEq` para afirmar igualdade desses tipos. Também precisará implementar `Debug` para imprimir os valores quando a asserção falhar. Como ambas as traits são deriváveis, como mencionado na Listagem 5-12 do Capítulo 5, isso costuma ser tão simples quanto adicionar a anotação `#[derive(PartialEq, Debug)]` à definição da struct ou enum. Consulte o [Apêndice C, “Traits deriváveis”](/livro/cap22-03-traits-derivaveis), para mais detalhes sobre essas e outras traits deriváveis.

### Adicionando mensagens de falha personalizadas

Você também pode adicionar uma mensagem personalizada para ser impressa com a mensagem de falha como argumentos opcionais das macros `assert!`, `assert_eq!` e `assert_ne!`. Quaisquer argumentos especificados após os argumentos obrigatórios são repassados à macro `format!` (discutida em Concatenando com `+` ou `format!` no Capítulo 8), então você pode passar uma string de formato que contém placeholders `{}` e valores para esses placeholders. Mensagens personalizadas são úteis para documentar o que uma asserção significa; quando um teste falha, você terá uma ideia melhor de qual é o problema no código.

Por exemplo, digamos que temos uma função que cumprimenta pessoas pelo nome e queremos testar que o nome que passamos à função aparece na saída:

**Arquivo: src/lib.rs**

```rust
pub fn greeting(name: &str) -> String {
    format!("Hello {name}!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

Os requisitos deste programa ainda não foram acordados, e temos bastante certeza de que o texto `Hello` no início da saudação mudará. Decidimos que não queremos ter de atualizar o teste quando os requisitos mudarem, então em vez de verificar igualdade exata com o valor retornado pela função `greeting`, apenas afirmaremos que a saída contém o texto do parâmetro de entrada.

Agora vamos introduzir um bug neste código mudando `greeting` para excluir `name` e ver como a falha de teste padrão parece:

**Arquivo: src/lib.rs**

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Executar este teste produz o seguinte:

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----

thread 'tests::greeting_contains_name' panicked at src/lib.rs:12:9:
assertion failed: result.contains("Carol")
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Este resultado só indica que a asserção falhou e em qual linha a asserção está. Uma mensagem de falha mais útil imprimiria o valor da função `greeting`. Vamos adicionar uma mensagem de falha personalizada composta de uma string de formato com um placeholder preenchido com o valor real que obtivemos da função `greeting`:

**Arquivo: src/lib.rs**

```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{result}`"
        );
    }
```

Agora, quando executarmos o teste, obteremos uma mensagem de erro mais informativa:

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----

thread 'tests::greeting_contains_name' panicked at src/lib.rs:12:9:
Greeting did not contain name, value was `Hello!`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Podemos ver o valor que realmente obtivemos na saída do teste, o que nos ajudaria a depurar o que aconteceu em vez do que esperávamos que acontecesse.

### Verificando pânico com `should_panic`

Além de verificar valores de retorno, é importante verificar que nosso código lida com condições de erro como esperamos. Por exemplo, considere o tipo `Guess` que criamos no Capítulo 9, na Listagem 9-13. Outro código que usa `Guess` depende da garantia de que instâncias de `Guess` conterão apenas valores entre 1 e 100. Podemos escrever um teste que garante que tentar criar uma instância de `Guess` com valor fora desse intervalo entra em pânico.

Fazemos isso adicionando o atributo `should_panic` à nossa função de teste. O teste passa se o código dentro da função entrar em pânico; o teste falha se o código dentro da função não entrar em pânico.

A Listagem 11-8 mostra um teste que verifica que as condições de erro de `Guess::new` ocorrem quando esperamos.

**Arquivo: src/lib.rs**

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<a id="listagem-11-8"></a>

[Listagem 11-8](#listagem-11-8): Testando que uma condição causará `panic!`

Colocamos o atributo `#[should_panic]` após o atributo `#[test]` e antes da função de teste a que se aplica. Vejamos o resultado quando este teste passa:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Parece bom! Agora vamos introduzir um bug no código removendo a condição de que a função `new` entrará em pânico se o valor for maior que 100:

**Arquivo: src/lib.rs**

```rust
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }
}
```

Quando executamos o teste da Listagem 11-8, ele falhará:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected at src/lib.rs:21:8

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Não obtemos uma mensagem muito útil neste caso, mas quando olhamos a função de teste, vemos que está anotada com `#[should_panic]`. A falha que obtivemos significa que o código na função de teste não causou pânico.

Testes que usam `should_panic` podem ser imprecisos. Um teste `should_panic` passaria mesmo se o teste entrasse em pânico por um motivo diferente do que esperávamos. Para tornar testes `should_panic` mais precisos, podemos adicionar um parâmetro opcional `expected` ao atributo `should_panic`. O test harness garantirá que a mensagem de falha contenha o texto fornecido. Por exemplo, considere o código modificado para `Guess` na Listagem 11-9, em que a função `new` entra em pânico com mensagens diferentes dependendo de o valor ser muito pequeno ou muito grande.

**Arquivo: src/lib.rs**

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {value}."
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {value}."
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<a id="listagem-11-9"></a>

[Listagem 11-9](#listagem-11-9): Testando `panic!` com mensagem de pânico contendo uma substring especificada

Este teste passará porque o valor que colocamos no parâmetro `expected` do atributo `should_panic` é uma substring da mensagem com a qual a função `Guess::new` entra em pânico. Poderíamos ter especificado a mensagem de pânico inteira que esperamos, que neste caso seria `Guess value must be less than or equal to 100, got 200`. O que você escolhe especificar depende de quanto da mensagem de pânico é única ou dinâmica e de quão preciso você quer que seu teste seja. Neste caso, uma substring da mensagem de pânico é suficiente para garantir que o código na função de teste executa o caso `else if value > 100`.

Para ver o que acontece quando um teste `should_panic` com mensagem `expected` falha, vamos novamente introduzir um bug no código trocando os corpos dos blocos `if value < 1` e `else if value > 100`:

**Arquivo: src/lib.rs**

```rust
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be less than or equal to 100, got {value}."
            );
        } else if value > 100 {
            panic!(
                "Guess value must be greater than or equal to 1, got {value}."
            );
        }

        Guess { value }
    }
}
```

Desta vez, quando executamos o teste `should_panic`, ele falhará:

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----

thread 'tests::greater_than_100' panicked at src/lib.rs:12:13:
Guess value must be greater than or equal to 1, got 200.
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
note: panic did not contain expected string
      panic message: "Guess value must be greater than or equal to 1, got 200."
 expected substring: "less than or equal to 100"

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

A mensagem de falha indica que este teste de fato entrou em pânico como esperávamos, mas a mensagem de pânico não incluiu a string esperada `less than or equal to 100`. A mensagem de pânico que obtivemos neste caso foi `Guess value must be greater than or equal to 1, got 200`. Agora podemos começar a descobrir onde está nosso bug!

### Usando `Result<T, E>` em testes

Todos os nossos testes até agora entraram em pânico quando falharam. Também podemos escrever testes que usam `Result<T, E>`! Aqui está o teste da Listagem 11-1, reescrito para usar `Result<T, E>` e retornar um `Err` em vez de entrar em pânico:

**Arquivo: src/lib.rs**

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() -> Result<(), String> {
        let result = add(2, 2);

        if result == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

A função `it_works` agora tem o tipo de retorno `Result<(), String>`. No corpo da função, em vez de chamar a macro `assert_eq!`, retornamos `Ok(())` quando o teste passa e um `Err` com um `String` dentro quando o teste falha.

Escrever testes para que retornem um `Result<T, E>` permite usar o operador `?` no corpo dos testes, o que pode ser uma forma conveniente de escrever testes que devem falhar se qualquer operação dentro deles retornar uma variante `Err`.

Você não pode usar a anotação `#[should_panic]` em testes que usam `Result<T, E>`. Para afirmar que uma operação retorna uma variante `Err`, _não_ use o operador `?` no valor `Result<T, E>`. Em vez disso, use `assert!(value.is_err())`.

Agora que você conhece várias formas de escrever testes, vamos ver o que acontece quando executamos nossos testes e explorar as diferentes opções que podemos usar com `cargo test`.
