---
title: "Organização de testes"
chapter_code: 11-03
slug: organizacao-de-testes
challenge_day: 14
reading_minutes: 13
---

# Organização de testes

Como mencionamos no início do capítulo, testar é uma disciplina complexa, e pessoas diferentes usam terminologias e formas de organização diferentes. A comunidade Rust pensa em testes a partir de duas categorias principais: testes unitários e testes de integração. _Testes unitários_ são pequenos e mais focados, testam um módulo por vez de forma isolada e podem testar interfaces privadas. _Testes de integração_ são totalmente externos à sua biblioteca e usam seu código da mesma forma que qualquer outro código externo usaria: usando apenas a interface pública e, potencialmente, exercitando vários módulos em um único teste.

Escrever os dois tipos de teste é importante para garantir que as partes da sua biblioteca façam o que você espera delas, tanto separadamente quanto em conjunto.

### Testes unitários

O objetivo dos testes unitários é testar cada unidade de código isoladamente do restante do código, para identificar rapidamente onde ele está ou não está funcionando como esperado. Você colocará os testes unitários no diretório _src_, em cada arquivo junto do código que eles testam. A convenção é criar um módulo chamado `tests` em cada arquivo para conter as funções de teste e anotar esse módulo com `cfg(test)`.

#### O módulo `tests` e `#[cfg(test)]`

A anotação `#[cfg(test)]` no módulo `tests` diz ao Rust para compilar e executar o código de teste apenas quando você executa `cargo test`, e não quando executa `cargo build`. Isso economiza tempo de compilação quando você só quer compilar a biblioteca e economiza espaço no artefato compilado resultante, porque os testes não são incluídos. Você verá que, como os testes de integração ficam em um diretório diferente, eles não precisam da anotação `#[cfg(test)]`. No entanto, como os testes unitários ficam nos mesmos arquivos que o código, você usará `#[cfg(test)]` para especificar que eles não devem ser incluídos no resultado compilado.

Lembre-se de que, quando geramos o novo projeto `adder` na primeira seção deste capítulo, o Cargo gerou este código para nós:

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

No módulo `tests` gerado automaticamente, o atributo `cfg` significa _configuration_ e diz ao Rust que o item seguinte só deve ser incluído quando uma determinada opção de configuração estiver presente. Neste caso, a opção de configuração é `test`, fornecida pelo Rust para compilar e executar testes. Ao usar o atributo `cfg`, o Cargo compila nosso código de teste apenas se executarmos ativamente os testes com `cargo test`. Isso inclui quaisquer funções auxiliares que possam estar dentro desse módulo, além das funções anotadas com `#[test]`.

#### Testes de funções privadas

Há um debate na comunidade de testes sobre se funções privadas devem ou não ser testadas diretamente, e outras linguagens tornam difícil ou impossível testar funções privadas. Independentemente da linha de pensamento sobre testes que você siga, as regras de privacidade do Rust permitem testar funções privadas. Considere o código da Listagem 11-12, com a função privada `internal_adder`.

**Arquivo: src/lib.rs**

```rust
pub fn add_two(a: u64) -> u64 {
    internal_adder(a, 2)
}

fn internal_adder(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        let result = internal_adder(2, 2);
        assert_eq!(result, 4);
    }
}
```

<a id="listagem-11-12"></a>

[Listagem 11-12](#listagem-11-12): Testando uma função privada

Observe que a função `internal_adder` não está marcada como `pub`. Testes são apenas código Rust, e o módulo `tests` é apenas mais um módulo. Como discutimos em [Caminhos para se referir a um item na árvore de módulos](/livro/cap07-03-caminhos-para-referenciar-um-item-na-arvore-de-modulos), itens em módulos filhos podem usar os itens de seus módulos ancestrais. Neste teste, trazemos todos os itens pertencentes ao módulo pai de `tests` para o escopo com `use super::*`, e então o teste pode chamar `internal_adder`. Se você não acha que funções privadas devem ser testadas, não há nada em Rust que obrigue você a fazer isso.

### Testes de integração

Em Rust, testes de integração são totalmente externos à sua biblioteca. Eles usam sua biblioteca da mesma forma que qualquer outro código usaria, o que significa que só podem chamar funções que fazem parte da API pública da biblioteca. O objetivo deles é testar se muitas partes da sua biblioteca funcionam corretamente juntas. Unidades de código que funcionam corretamente por conta própria podem apresentar problemas quando integradas, então a cobertura de testes do código integrado também é importante. Para criar testes de integração, primeiro você precisa de um diretório _tests_.

#### O diretório _tests_

Criamos um diretório _tests_ no nível superior do diretório do nosso projeto, ao lado de _src_. O Cargo sabe que deve procurar arquivos de teste de integração nesse diretório. Podemos então criar quantos arquivos de teste quisermos, e o Cargo compilará cada um deles como uma crate individual.

Vamos criar um teste de integração. Com o código da Listagem 11-12 ainda no arquivo _src/lib.rs_, crie um diretório _tests_ e um novo arquivo chamado _tests/integration_test.rs_. Sua estrutura de diretórios deve ficar assim:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Insira o código da Listagem 11-13 no arquivo _tests/integration_test.rs_.

**Arquivo: tests/integration_test.rs**

```rust
use adder::add_two;

#[test]
fn it_adds_two() {
    let result = add_two(2);
    assert_eq!(result, 4);
}
```

<a id="listagem-11-13"></a>

[Listagem 11-13](#listagem-11-13): Um teste de integração de uma função na crate `adder`

Cada arquivo no diretório _tests_ é uma crate separada, então precisamos trazer nossa biblioteca para o escopo de cada crate de teste. Por esse motivo, adicionamos `use adder::add_two;` no topo do código, algo que não precisávamos fazer nos testes unitários.

Não precisamos anotar nenhum código em _tests/integration_test.rs_ com `#[cfg(test)]`. O Cargo trata o diretório _tests_ de forma especial e compila os arquivos nesse diretório apenas quando executamos `cargo test`. Execute `cargo test` agora:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests src/lib.rs (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

As três seções da saída incluem os testes unitários, o teste de integração e os testes de documentação. Observe que, se qualquer teste em uma seção falhar, as seções seguintes não serão executadas. Por exemplo, se um teste unitário falhar, não haverá saída para os testes de integração nem para os testes de documentação, porque esses testes só serão executados se todos os testes unitários passarem.

A primeira seção, referente aos testes unitários, é a mesma que temos visto: uma linha para cada teste unitário (um chamado `internal`, que adicionamos na Listagem 11-12) e então uma linha de resumo para os testes unitários.

A seção dos testes de integração começa com a linha `Running tests/integration_test.rs`. Em seguida, há uma linha para cada função de teste nesse teste de integração e uma linha de resumo para os resultados do teste de integração logo antes do início da seção `Doc-tests adder`.

Cada arquivo de teste de integração tem sua própria seção, então, se adicionarmos mais arquivos ao diretório _tests_, haverá mais seções de testes de integração.

Ainda podemos executar uma função específica de teste de integração especificando o nome da função de teste como argumento para `cargo test`. Para executar todos os testes em um arquivo específico de teste de integração, use o argumento `--test` de `cargo test` seguido do nome do arquivo:

```console
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Esse comando executa apenas os testes no arquivo _tests/integration_test.rs_.

#### Submódulos em testes de integração

À medida que você adiciona mais testes de integração, talvez queira criar mais arquivos no diretório _tests_ para ajudar a organizá-los; por exemplo, você pode agrupar as funções de teste pela funcionalidade que elas testam. Como mencionado antes, cada arquivo no diretório _tests_ é compilado como sua própria crate separada, o que é útil para criar escopos separados que imitam mais de perto a forma como usuários finais usarão sua crate. No entanto, isso significa que os arquivos no diretório _tests_ não compartilham o mesmo comportamento dos arquivos em _src_, como você aprendeu no Capítulo 7 ao ver como separar código em módulos e arquivos.

O comportamento diferente dos arquivos do diretório _tests_ fica mais perceptível quando você tem um conjunto de funções auxiliares para usar em vários arquivos de teste de integração e tenta seguir os passos da seção [Separando módulos em arquivos diferentes](/livro/cap07-05-separando-modulos-em-arquivos-diferentes), do Capítulo 7, para extraí-las em um módulo comum. Por exemplo, se criarmos _tests/common.rs_ e colocarmos nele uma função chamada `setup`, podemos adicionar a `setup` algum código que queremos chamar a partir de várias funções de teste em vários arquivos de teste:

**Arquivo: tests/common.rs**

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

Quando executarmos os testes novamente, veremos uma nova seção na saída dos testes para o arquivo _common.rs_, mesmo que esse arquivo não contenha funções de teste e mesmo que não tenhamos chamado a função `setup` de nenhum lugar:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Ter `common` aparecendo nos resultados dos testes com `running 0 tests` exibido para ele não é o que queríamos. Queríamos apenas compartilhar algum código com os outros arquivos de teste de integração. Para evitar que `common` apareça na saída dos testes, em vez de criar _tests/common.rs_, criaremos _tests/common/mod.rs_. O diretório do projeto agora fica assim:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Essa é a convenção de nomenclatura mais antiga que o Rust também entende e que mencionamos em “Caminhos de arquivo alternativos”, no Capítulo 7. Nomear o arquivo dessa forma diz ao Rust para não tratar o módulo `common` como um arquivo de teste de integração. Quando movemos o código da função `setup` para _tests/common/mod.rs_ e excluímos o arquivo _tests/common.rs_, a seção na saída dos testes não aparecerá mais. Arquivos em subdiretórios do diretório _tests_ não são compilados como crates separadas nem têm seções na saída dos testes.

Depois de criarmos _tests/common/mod.rs_, podemos usá-lo como módulo a partir de qualquer arquivo de teste de integração. Aqui está um exemplo de chamada da função `setup` a partir do teste `it_adds_two` em _tests/integration_test.rs_:

**Arquivo: tests/integration_test.rs**

```rust
use adder::add_two;

mod common;

#[test]
fn it_adds_two() {
    common::setup();

    let result = add_two(2);
    assert_eq!(result, 4);
}
```

Observe que a declaração `mod common;` é a mesma declaração de módulo que demonstramos na Listagem 7-21. Então, na função de teste, podemos chamar `common::setup()`.

#### Testes de integração para crates binárias

Se nosso projeto for uma crate binária que contém apenas um arquivo _src/main.rs_ e não tiver um arquivo _src/lib.rs_, não poderemos criar testes de integração no diretório _tests_ e trazer funções definidas no arquivo _src/main.rs_ para o escopo com uma instrução `use`. Apenas crates de biblioteca expõem funções que outras crates podem usar; crates binárias são feitas para serem executadas por conta própria.

Essa é uma das razões pelas quais projetos Rust que fornecem um binário costumam ter um arquivo _src/main.rs_ simples, que chama a lógica que vive no arquivo _src/lib.rs_. Usando essa estrutura, os testes de integração _podem_ testar a crate de biblioteca com `use` para disponibilizar a funcionalidade importante. Se a funcionalidade importante funcionar, a pequena quantidade de código no arquivo _src/main.rs_ também funcionará, e essa pequena quantidade de código não precisa ser testada.

## Resumo

Os recursos de teste do Rust fornecem uma forma de especificar como o código deve funcionar para garantir que ele continue se comportando como você espera, mesmo quando você fizer alterações. Testes unitários exercitam diferentes partes de uma biblioteca separadamente e podem testar detalhes privados de implementação. Testes de integração verificam se muitas partes da biblioteca funcionam corretamente juntas e usam a API pública da biblioteca para testar o código da mesma forma que um código externo o usaria. Embora o sistema de tipos e as regras de ownership do Rust ajudem a prevenir alguns tipos de bugs, testes ainda são importantes para reduzir bugs de lógica relacionados ao comportamento esperado do seu código.

Vamos combinar o conhecimento que você aprendeu neste capítulo e nos capítulos anteriores para trabalhar em um projeto!
