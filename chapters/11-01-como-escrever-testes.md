---
title: "Como escrever testes"
chapter_code: 11-01
slug: como-escrever-testes
---

# Como escrever testes

_Testes_ são funções Rust que verificam se o código que não é de teste está funcionando da maneira esperada. O corpo de uma função de teste normalmente realiza estas três ações:

- Configurar quaisquer dados ou estado necessários.
- Executar o código que você quer testar.
- Verificar se os resultados são os esperados.

Vamos conhecer os recursos que o Rust fornece especificamente para escrever testes que realizam essas ações, incluindo o atributo `test`, algumas macros e o atributo `should_panic`.

### Estruturando funções de teste

Em sua forma mais simples, um teste em Rust é uma função anotada com o atributo `test`. Atributos são metadados sobre partes do código Rust; um exemplo é o atributo `derive` que usamos com structs no Capítulo 5. Para transformar uma função em uma função de teste, adicione `#[test]` na linha anterior a `fn`. Quando você executa seus testes com o comando `cargo test`, o Rust compila um binário executor de testes que executa as funções anotadas e informa se cada função de teste passou ou falhou.

Sempre que criamos um novo projeto de biblioteca com o Cargo, um módulo de teste com uma função de teste é gerado automaticamente para nós. Esse módulo oferece um modelo para escrever seus testes, para que você não precise procurar a estrutura e a sintaxe exatas toda vez que iniciar um novo projeto. Você pode adicionar quantas funções de teste e quantos módulos de teste adicionais quiser!

Vamos explorar alguns aspectos de como os testes funcionam experimentando com o teste-modelo antes de realmente testar qualquer código. Depois, escreveremos alguns testes mais próximos do uso real, que chamam código que escrevemos e verificam se seu comportamento está correto.

Vamos criar um novo projeto de biblioteca chamado `adder`, que somará dois números:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

O conteúdo do arquivo _src/lib.rs_ na sua biblioteca `adder` deve ficar parecido com a Listagem 11-1.

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

O arquivo começa com uma função `add` de exemplo para que tenhamos algo para testar.

Por enquanto, vamos nos concentrar apenas na função `it_works`. Observe a anotação `#[test]`: esse atributo indica que esta é uma função de teste, então o executor de testes sabe que deve tratá-la como um teste. Também podemos ter funções que não são testes no módulo `tests` para ajudar a configurar cenários comuns ou realizar operações comuns, por isso sempre precisamos indicar quais funções são testes.

O corpo da função de exemplo usa a macro `assert_eq!` para verificar que `result`, que contém o resultado de chamar `add` com 2 e 2, é igual a 4. Essa asserção serve como exemplo do formato de um teste típico. Vamos executá-lo para ver se esse teste passa.

O comando `cargo test` executa todos os testes do nosso projeto, como mostrado na Listagem 11-2.

```console
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

O Cargo compilou e executou o teste. Vemos a linha `running 1 test`. A linha seguinte mostra o nome da função de teste gerada, chamada `tests::it_works`, e informa que o resultado da execução desse teste é `ok`. O resumo geral `test result: ok.` significa que todos os testes passaram, e a parte que diz `1 passed; 0 failed` totaliza o número de testes que passaram ou falharam.

É possível marcar um teste como ignorado para que ele não seja executado em uma determinada ocasião; abordaremos isso na seção “Ignorando testes a menos que sejam solicitados especificamente”, mais adiante neste capítulo. Como não fizemos isso aqui, o resumo mostra `0 ignored`. Também podemos passar um argumento ao comando `cargo test` para executar apenas testes cujo nome corresponda a uma string; isso é chamado de _filtragem_, e abordaremos esse recurso na seção “Executando um subconjunto de testes por nome”. Aqui, não filtramos os testes executados, então o fim do resumo mostra `0 filtered out`.

A estatística `0 measured` é para testes de benchmark, que medem desempenho. No momento em que escrevemos, testes de benchmark estão disponíveis apenas no Rust nightly. Consulte a [documentação sobre testes de benchmark](https://doc.rust-lang.org/nightly/unstable-book/library-features/test.html) para saber mais.

A próxima parte da saída dos testes, começando em `Doc-tests adder`, mostra os resultados de quaisquer testes de documentação. Ainda não temos testes de documentação, mas o Rust pode compilar qualquer exemplo de código que apareça na documentação da nossa API. Esse recurso ajuda a manter sua documentação e seu código sincronizados! Discutiremos como escrever testes de documentação na seção “Comentários de documentação como testes” do Capítulo 14. Por enquanto, vamos ignorar a saída `Doc-tests`.

Vamos começar a personalizar o teste para nossas próprias necessidades. Primeiro, mude o nome da função `it_works` para um nome diferente, como `exploration`, assim:

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

```console
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

Agora adicionaremos outro teste, mas desta vez faremos um teste que falha! Testes falham quando algo na função de teste entra em pânico. Cada teste é executado em uma nova thread e, quando a thread principal percebe que uma thread de teste morreu, o teste é marcado como falho. No Capítulo 9, falamos que a forma mais simples de entrar em pânico é chamar a macro `panic!`. Insira o novo teste como uma função chamada `another`, para que seu arquivo _src/lib.rs_ fique como na Listagem 11-3.

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

```console
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

Em vez de `ok`, a linha `test tests::another` mostra `FAILED`. Duas novas seções aparecem entre os resultados individuais e o resumo: a primeira exibe o motivo detalhado de cada falha de teste. Neste caso, obtemos os detalhes de que `tests::another` falhou porque entrou em pânico com a mensagem `Make this test fail` na linha 17 do arquivo _src/lib.rs_. A seção seguinte lista apenas os nomes de todos os testes que falharam, o que é útil quando há muitos testes e muita saída detalhada de falhas. Podemos usar o nome de um teste que falhou para executar apenas esse teste e depurá-lo com mais facilidade; falaremos mais sobre formas de executar testes na seção “Controlando como os testes são executados”.

A linha de resumo aparece no final: no geral, nosso resultado de teste é `FAILED`. Tivemos um teste que passou e um teste que falhou.

Agora que você viu como os resultados dos testes aparecem em diferentes cenários, vamos conhecer algumas macros além de `panic!` que são úteis em testes.

### Verificando resultados com `assert!`

A macro `assert!`, fornecida pela biblioteca padrão, é útil quando você quer garantir que alguma condição em um teste seja avaliada como `true`. Passamos para a macro `assert!` um argumento que é avaliado como um booleano. Se o valor for `true`, nada acontece e o teste passa. Se o valor for `false`, a macro `assert!` chama `panic!` para fazer o teste falhar. Usar a macro `assert!` nos ajuda a verificar se nosso código está funcionando da maneira que pretendemos.

No Capítulo 5, na Listagem 5-15, usamos uma struct `Rectangle` e um método `can_hold`, que repetimos aqui na Listagem 11-5. Vamos colocar esse código no arquivo _src/lib.rs_ e então escrever alguns testes para ele usando a macro `assert!`.

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

O método `can_hold` retorna um booleano, o que significa que ele é um caso perfeito para usar a macro `assert!`. Na Listagem 11-6, escrevemos um teste que exercita o método `can_hold` criando uma instância de `Rectangle` com largura 8 e altura 7 e verificando que ela consegue conter outra instância de `Rectangle` com largura 5 e altura 1.

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

Observe a linha `use super::*;` dentro do módulo `tests`. O módulo `tests` é um módulo comum que segue as regras de visibilidade usuais que abordamos no Capítulo 7, na seção “Caminhos para se referir a um item na árvore de módulos”. Como o módulo `tests` é um módulo interno, precisamos trazer o código sob teste, que está no módulo externo, para o escopo do módulo interno. Usamos um glob aqui, então qualquer coisa que definirmos no módulo externo fica disponível para esse módulo `tests`.

Chamamos nosso teste de `larger_can_hold_smaller` e criamos as duas instâncias de `Rectangle` de que precisamos. Depois, chamamos a macro `assert!` e passamos o resultado da chamada `larger.can_hold(&smaller)`. Essa expressão deve retornar `true`, então nosso teste deve passar. Vamos conferir!

```console
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

Passou! Vamos adicionar outro teste, desta vez verificando que um retângulo menor não pode conter um maior:

**Arquivo: src/lib.rs**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

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
}
```

Como o resultado correto da função `can_hold` neste caso é `false`, precisamos negar esse resultado antes de passá-lo para a macro `assert!`. Como consequência, nosso teste passará se `can_hold` retornar `false`:

```console
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

Dois testes passando! Agora vamos ver o que acontece com os resultados dos testes quando introduzimos um bug no código. Mudaremos a implementação do método `can_hold` substituindo o sinal de maior que (`>`) por um sinal de menor que (`<`) ao comparar as larguras:

**Arquivo: src/lib.rs (Este código não produz o comportamento desejado.)**

```rust
// --snip--
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}
```

Executar os testes agora produz o seguinte:

```console
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

Nossos testes encontraram o bug! Como `larger.width` é `8` e `smaller.width` é `5`, a comparação das larguras em `can_hold` agora retorna `false`: 8 não é menor que 5.

### Testando igualdade com `assert_eq!` e `assert_ne!`

Uma forma comum de verificar uma funcionalidade é testar a igualdade entre o resultado do código sob teste e o valor que você espera que o código retorne. Você poderia fazer isso usando a macro `assert!` e passando uma expressão com o operador `==`. No entanto, esse é um teste tão comum que a biblioteca padrão fornece um par de macros, `assert_eq!` e `assert_ne!`, para realizá-lo de maneira mais conveniente. Essas macros comparam dois argumentos quanto à igualdade ou à desigualdade, respectivamente. Elas também imprimem os dois valores se a asserção falhar, o que facilita entender por que o teste falhou; em contraste, a macro `assert!` apenas indica que recebeu o valor `false` para a expressão `==`, sem imprimir os valores que levaram a esse resultado.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao seu parâmetro e então testamos essa função usando a macro `assert_eq!`.

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

```console
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

Criamos uma variável chamada `result` que contém o resultado de chamar `add_two(2)`. Depois, passamos `result` e `4` como argumentos para a macro `assert_eq!`. A linha de saída desse teste é `test tests::it_adds_two ... ok`, e o texto `ok` indica que nosso teste passou!

Vamos introduzir um bug no código para ver como `assert_eq!` aparece quando falha. Mude a implementação da função `add_two` para adicionar `3` em vez de `2`:

**Arquivo: src/lib.rs (Este código não produz o comportamento desejado.)**

```rust
pub fn add_two(a: u64) -> u64 {
    a + 3
}
```

Execute os testes novamente:

```console
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

Nosso teste encontrou o bug! O teste `tests::it_adds_two` falhou, e a mensagem nos diz que a asserção que falhou foi `left == right` e mostra quais são os valores de `left` e `right`. Essa mensagem nos ajuda a começar a depurar: o argumento `left`, onde tínhamos o resultado de chamar `add_two(2)`, foi `5`, mas o argumento `right` foi `4`. Você pode imaginar que isso seria especialmente útil quando há muitos testes em execução.

Observe que, em algumas linguagens e frameworks de teste, os parâmetros das funções de asserção de igualdade são chamados de `expected` e `actual`, e a ordem em que especificamos os argumentos importa. Em Rust, porém, eles são chamados de `left` e `right`, e a ordem em que especificamos o valor que esperamos e o valor produzido pelo código não importa. Poderíamos escrever a asserção neste teste como `assert_eq!(4, result)`, o que resultaria na mesma mensagem de falha exibindo `` assertion `left == right` failed ``.

A macro `assert_ne!` passa se os dois valores que fornecemos a ela não forem iguais e falha se forem iguais. Essa macro é mais útil nos casos em que não temos certeza de qual será um valor, mas sabemos qual valor ele definitivamente _não_ deve ser. Por exemplo, se estamos testando uma função que tem a garantia de alterar sua entrada de alguma forma, mas a maneira como a entrada é alterada depende do dia da semana em que executamos os testes, talvez a melhor asserção seja verificar que a saída da função não é igual à entrada.

Por baixo dos panos, as macros `assert_eq!` e `assert_ne!` usam os operadores `==` e `!=`, respectivamente. Quando as asserções falham, essas macros imprimem seus argumentos usando a formatação de debug, o que significa que os valores comparados devem implementar as traits `PartialEq` e `Debug`. Todos os tipos primitivos e a maioria dos tipos da biblioteca padrão implementam essas traits. Para structs e enums que você define, será necessário implementar `PartialEq` para verificar a igualdade desses tipos. Você também precisará implementar `Debug` para imprimir os valores quando a asserção falhar. Como ambas são traits deriváveis, conforme mencionado na Listagem 5-12 do Capítulo 5, isso normalmente é tão simples quanto adicionar a anotação `#[derive(PartialEq, Debug)]` à definição da struct ou do enum. Consulte o [Apêndice C, “Traits deriváveis”](/livro/cap22-03-traits-derivaveis), para mais detalhes sobre essas e outras traits deriváveis.

### Adicionando mensagens de falha personalizadas

Você também pode adicionar uma mensagem personalizada para ser impressa junto com a mensagem de falha, usando argumentos opcionais nas macros `assert!`, `assert_eq!` e `assert_ne!`. Quaisquer argumentos especificados depois dos argumentos obrigatórios são repassados para a macro `format!` (discutida em “Concatenando com `+` ou `format!`” no Capítulo 8), então você pode passar uma string de formato que contenha placeholders `{}` e valores para preencher esses placeholders. Mensagens personalizadas são úteis para documentar o que uma asserção significa; quando um teste falhar, você terá uma ideia melhor de qual é o problema no código.

Por exemplo, digamos que temos uma função que cumprimenta pessoas pelo nome e queremos testar se o nome que passamos para a função aparece na saída:

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

Os requisitos desse programa ainda não foram definidos, e temos bastante certeza de que o texto `Hello` no início da saudação mudará. Decidimos que não queremos ter que atualizar o teste quando os requisitos mudarem, então, em vez de verificar igualdade exata com o valor retornado pela função `greeting`, vamos apenas verificar que a saída contém o texto do parâmetro de entrada.

Agora vamos introduzir um bug nesse código alterando `greeting` para excluir `name` e ver como é a falha de teste padrão:

**Arquivo: src/lib.rs (Este código não produz o comportamento desejado.)**

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Executar este teste produz o seguinte:

```console
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

Esse resultado apenas indica que a asserção falhou e mostra em qual linha ela está. Uma mensagem de falha mais útil imprimiria o valor retornado pela função `greeting`. Vamos adicionar uma mensagem de falha personalizada composta por uma string de formato com um placeholder preenchido pelo valor real que obtivemos da função `greeting`:

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

Agora, quando executarmos o teste, receberemos uma mensagem de erro mais informativa:

```console
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

Podemos ver na saída do teste o valor que realmente obtivemos, o que nos ajudaria a depurar o que aconteceu em vez de ficarmos apenas com o que esperávamos que acontecesse.

### Verificando pânicos com `should_panic`

Além de verificar valores de retorno, é importante verificar se nosso código lida com condições de erro da maneira que esperamos. Por exemplo, considere o tipo `Guess` que criamos no Capítulo 9, na Listagem 9-13. Outros códigos que usam `Guess` dependem da garantia de que instâncias de `Guess` conterão apenas valores entre 1 e 100. Podemos escrever um teste que garante que tentar criar uma instância de `Guess` com um valor fora desse intervalo causará um pânico.

Fazemos isso adicionando o atributo `should_panic` à nossa função de teste. O teste passa se o código dentro da função entrar em pânico; o teste falha se o código dentro da função não entrar em pânico.

A Listagem 11-8 mostra um teste que verifica se as condições de erro de `Guess::new` acontecem quando esperamos que aconteçam.

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

[Listagem 11-8](#listagem-11-8): Testando que uma condição causará um `panic!`

Colocamos o atributo `#[should_panic]` depois do atributo `#[test]` e antes da função de teste à qual ele se aplica. Vejamos o resultado quando esse teste passa:

```console
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

Tudo certo! Agora vamos introduzir um bug no código removendo a condição que faz a função `new` entrar em pânico se o valor for maior que 100:

**Arquivo: src/lib.rs (Este código não produz o comportamento desejado.)**

```rust
// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }
}
```

Quando executarmos o teste da Listagem 11-8, ele falhará:

```console
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

Não recebemos uma mensagem muito útil nesse caso, mas, quando olhamos para a função de teste, vemos que ela está anotada com `#[should_panic]`. A falha que obtivemos significa que o código na função de teste não causou um pânico.

Testes que usam `should_panic` podem ser imprecisos. Um teste `should_panic` passaria mesmo se o teste entrasse em pânico por um motivo diferente daquele que esperávamos. Para tornar testes `should_panic` mais precisos, podemos adicionar um parâmetro opcional `expected` ao atributo `should_panic`. O sistema de testes garantirá que a mensagem de falha contenha o texto fornecido. Por exemplo, considere o código modificado para `Guess` na Listagem 11-9, em que a função `new` entra em pânico com mensagens diferentes dependendo de o valor ser pequeno demais ou grande demais.

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

Esse teste passará porque o valor que colocamos no parâmetro `expected` do atributo `should_panic` é uma substring da mensagem com a qual a função `Guess::new` entra em pânico. Poderíamos ter especificado a mensagem de pânico inteira que esperamos, que nesse caso seria `Guess value must be less than or equal to 100, got 200`. O que você escolhe especificar depende de quanto da mensagem de pânico é único ou dinâmico e de quão preciso você quer que seu teste seja. Neste caso, uma substring da mensagem de pânico é suficiente para garantir que o código na função de teste execute o caso `else if value > 100`.

Para ver o que acontece quando um teste `should_panic` com uma mensagem `expected` falha, vamos novamente introduzir um bug no código trocando os corpos dos blocos `if value < 1` e `else if value > 100`:

**Arquivo: src/lib.rs (Este código não produz o comportamento desejado.)**

```rust
// --snip--
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

Desta vez, quando executarmos o teste `should_panic`, ele falhará:

```console
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

A mensagem de falha indica que esse teste de fato entrou em pânico como esperávamos, mas que a mensagem de pânico não incluiu a string esperada `less than or equal to 100`. A mensagem de pânico que recebemos nesse caso foi `Guess value must be greater than or equal to 1, got 200`. Agora podemos começar a descobrir onde está o bug!

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

A função `it_works` agora tem o tipo de retorno `Result<(), String>`. No corpo da função, em vez de chamar a macro `assert_eq!`, retornamos `Ok(())` quando o teste passa e um `Err` contendo uma `String` quando o teste falha.

Escrever testes de modo que eles retornem um `Result<T, E>` permite usar o operador `?` no corpo dos testes, o que pode ser uma forma conveniente de escrever testes que devem falhar se qualquer operação dentro deles retornar uma variante `Err`.

Você não pode usar a anotação `#[should_panic]` em testes que usam `Result<T, E>`. Para verificar que uma operação retorna uma variante `Err`, _não_ use o operador `?` no valor `Result<T, E>`. Em vez disso, use `assert!(value.is_err())`.

Agora que você conhece várias formas de escrever testes, vamos ver o que acontece quando os executamos e explorar as diferentes opções que podemos usar com `cargo test`.
