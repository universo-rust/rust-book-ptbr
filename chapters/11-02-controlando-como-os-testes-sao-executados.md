---
title: "Controlando como os testes são executados"
chapter_code: 11-02
slug: controlando-como-os-testes-sao-executados
---

# Controlando como os testes são executados

Assim como `cargo run` compila seu código e depois executa o binário resultante, `cargo test` compila seu código em modo de teste e executa o binário de teste resultante. O comportamento padrão do binário produzido por `cargo test` é executar todos os testes em paralelo e capturar a saída gerada durante a execução deles, impedindo que essa saída seja exibida e facilitando a leitura das informações relacionadas aos resultados dos testes. No entanto, você pode especificar opções de linha de comando para alterar esse comportamento padrão.

Algumas opções de linha de comando são destinadas a `cargo test`, e outras são destinadas ao binário de teste resultante. Para separar esses dois tipos de argumento, liste primeiro os argumentos que vão para `cargo test`, depois o separador `--` e, em seguida, os argumentos que vão para o binário de teste. Executar `cargo test --help` exibe as opções que você pode usar com `cargo test`, e executar `cargo test -- --help` exibe as opções que você pode usar depois do separador. Essas opções também estão documentadas na [seção “Tests” do _The `rustc` Book_](https://doc.rust-lang.org/rustc/tests/index.html).

### Executando testes em paralelo ou em sequência

Quando você executa vários testes, por padrão eles são executados em paralelo usando threads, o que significa que terminam mais rápido e você recebe feedback mais cedo. Como os testes são executados ao mesmo tempo, você precisa garantir que eles não dependam uns dos outros nem de qualquer estado compartilhado, incluindo um ambiente compartilhado, como o diretório de trabalho atual ou variáveis de ambiente.

Por exemplo, digamos que cada um dos seus testes execute algum código que cria um arquivo em disco chamado _test-output.txt_ e escreve alguns dados nesse arquivo. Depois, cada teste lê os dados desse arquivo e verifica se ele contém um determinado valor, diferente em cada teste. Como os testes são executados ao mesmo tempo, um teste pode sobrescrever o arquivo no intervalo entre o momento em que outro teste escreve no arquivo e o momento em que ele o lê. O segundo teste então falhará, não porque o código esteja incorreto, mas porque os testes interferiram uns nos outros durante a execução em paralelo. Uma solução é garantir que cada teste escreva em um arquivo diferente; outra solução é executar os testes um de cada vez.

Se você não quiser executar os testes em paralelo, ou se quiser um controle mais refinado sobre o número de threads usadas, pode enviar a flag `--test-threads` e o número de threads desejado para o binário de teste. Veja o exemplo a seguir:

```console
$ cargo test -- --test-threads=1
```

Definimos o número de threads de teste como `1`, dizendo ao programa para não usar paralelismo. Executar os testes com uma única thread levará mais tempo do que executá-los em paralelo, mas os testes não interferirão uns nos outros se compartilharem estado.

### Exibindo a saída das funções

Por padrão, se um teste passa, a biblioteca de testes do Rust captura tudo que foi impresso na saída padrão. Por exemplo, se chamarmos `println!` em um teste e esse teste passar, não veremos a saída de `println!` no terminal; veremos apenas a linha indicando que o teste passou. Se um teste falhar, veremos o que foi impresso na saída padrão junto com o restante da mensagem de falha.

Como exemplo, a Listagem 11-10 tem uma função boba que imprime o valor de seu parâmetro e retorna 10, além de um teste que passa e outro que falha.

**Arquivo: src/lib.rs (Este código entra em pânico!)**

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {a}");
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(value, 10);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(value, 5);
    }
}
```

<a id="listagem-11-10"></a>

[Listagem 11-10](#listagem-11-10): Testes para uma função que chama `println!`

Quando executamos esses testes com `cargo test`, veremos a seguinte saída:

```console
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8

thread 'tests::this_test_will_fail' panicked at src/lib.rs:19:9:
assertion `left == right` failed
  left: 10
 right: 5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Observe que, em nenhum ponto dessa saída, vemos `I got the value 4`, que é impresso quando o teste que passa é executado. Essa saída foi capturada. A saída do teste que falhou, `I got the value 8`, aparece na seção do resumo dos testes, que também mostra a causa da falha.

Se quisermos ver também os valores impressos pelos testes que passam, podemos dizer ao Rust para mostrar a saída dos testes bem-sucedidos com `--show-output`:

```console
$ cargo test -- --show-output
```

Quando executamos novamente os testes da Listagem 11-10 com a flag `--show-output`, vemos a seguinte saída:

```console
$ cargo test -- --show-output
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8

thread 'tests::this_test_will_fail' panicked at src/lib.rs:19:9:
assertion `left == right` failed
  left: 10
 right: 5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

### Executando um subconjunto de testes por nome

Executar uma suíte de testes completa às vezes pode levar bastante tempo. Se você está trabalhando em código de uma área específica, talvez queira executar apenas os testes relacionados a esse código. Você pode escolher quais testes executar passando para `cargo test`, como argumento, o nome ou os nomes dos testes que deseja executar.

Para demonstrar como executar um subconjunto de testes, primeiro criaremos três testes para nossa função `add_two`, como mostrado na Listagem 11-11, e então escolheremos quais deles executar.

**Arquivo: src/lib.rs**

```rust
pub fn add_two(a: u64) -> u64 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        let result = add_two(2);
        assert_eq!(result, 4);
    }

    #[test]
    fn add_three_and_two() {
        let result = add_two(3);
        assert_eq!(result, 5);
    }

    #[test]
    fn one_hundred() {
        let result = add_two(100);
        assert_eq!(result, 102);
    }
}
```

<a id="listagem-11-11"></a>

[Listagem 11-11](#listagem-11-11): Três testes com três nomes diferentes

Se executarmos os testes sem passar argumentos, como vimos antes, todos eles serão executados em paralelo:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 3 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

#### Executando um único teste

Podemos passar o nome de qualquer função de teste para `cargo test` a fim de executar apenas esse teste:

```console
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

Apenas o teste com o nome `one_hundred` foi executado; os outros dois testes não correspondiam a esse nome. A saída dos testes nos informa que havia mais testes que não foram executados exibindo `2 filtered out` no final.

Não podemos especificar os nomes de vários testes dessa forma; apenas o primeiro valor fornecido a `cargo test` será usado. Mas há uma forma de executar vários testes.

#### Filtrando para executar vários testes

Podemos especificar parte de um nome de teste, e qualquer teste cujo nome corresponda a esse valor será executado. Por exemplo, como dois dos nomes dos nossos testes contêm `add`, podemos executar esses dois rodando `cargo test add`:

```console
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

Esse comando executou todos os testes com `add` no nome e filtrou o teste chamado `one_hundred`. Observe também que o módulo em que um teste aparece se torna parte do nome do teste, então podemos executar todos os testes de um módulo filtrando pelo nome do módulo.

### Ignorando testes a menos que sejam solicitados especificamente

Às vezes, alguns testes específicos podem ser muito demorados para executar, então talvez você queira excluí-los da maioria das execuções de `cargo test`. Em vez de listar como argumentos todos os testes que você quer executar, você pode anotar os testes demorados com o atributo `ignore` para excluí-los, como mostrado aqui:

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

    #[test]
    #[ignore]
    fn expensive_test() {
        // code that takes an hour to run
    }
}
```

Depois de `#[test]`, adicionamos a linha `#[ignore]` ao teste que queremos excluir. Agora, quando executamos nossos testes, `it_works` é executado, mas `expensive_test` não:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::expensive_test ... ignored
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

A função `expensive_test` é listada como `ignored`. Se quisermos executar apenas os testes ignorados, podemos usar `cargo test -- --ignored`:

```console
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Ao controlar quais testes são executados, você pode garantir que os resultados de `cargo test` sejam retornados rapidamente. Quando estiver em um ponto em que faz sentido verificar os resultados dos testes `ignored` e tiver tempo para esperar por eles, execute `cargo test -- --ignored`. Se quiser executar todos os testes, ignorados ou não, execute `cargo test -- --include-ignored`.
