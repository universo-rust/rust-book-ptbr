---
title: "Controlando como os testes são executados"
chapter_code: 11-02
slug: controlando-como-os-testes-sao-executados
---

# Controlando Como os Testes São Executados

Assim como `cargo run` compila seu código e depois executa o binário resultante, `cargo test` compila seu código em modo de teste e executa o binário de teste resultante. O comportamento padrão do binário produzido por `cargo test` é executar todos os testes em paralelo e capturar a saída gerada durante a execução dos testes, impedindo que a saída seja exibida e facilitando a leitura da saída relacionada aos resultados dos testes. Você pode, porém, especificar opções de linha de comando para mudar este comportamento padrão.

Algumas opções de linha de comando vão para `cargo test`, e outras vão para o binário de teste resultante. Para separar esses dois tipos de argumentos, você lista os argumentos que vão para `cargo test`, seguidos do separador `--` e depois os que vão para o binário de teste. Executar `cargo test --help` exibe as opções que você pode usar com `cargo test`, e executar `cargo test -- --help` exibe as opções que você pode usar após o separador. Essas opções também estão documentadas na [seção “Tests” do _The `rustc` Book_](https://doc.rust-lang.org/rustc/tests/index.html).

### Executando testes em paralelo ou em sequência

Quando você executa vários testes, por padrão eles executam em paralelo usando threads, o que significa que terminam mais rápido e você recebe feedback mais cedo. Como os testes executam ao mesmo tempo, você deve garantir que seus testes não dependam uns dos outros nem de qualquer estado compartilhado, incluindo um ambiente compartilhado, como o diretório de trabalho atual ou variáveis de ambiente.

Por exemplo, digamos que cada um dos seus testes executa algum código que cria um arquivo em disco chamado _test-output.txt_ e escreve alguns dados nesse arquivo. Depois, cada teste lê os dados nesse arquivo e afirma que o arquivo contém um valor particular, que é diferente em cada teste. Como os testes executam ao mesmo tempo, um teste pode sobrescrever o arquivo no intervalo entre outro teste escrever e ler o arquivo. O segundo teste então falhará, não porque o código esteja incorreto, mas porque os testes interferiram uns nos outros ao executar em paralelo. Uma solução é garantir que cada teste escreva em um arquivo diferente; outra solução é executar os testes um de cada vez.

Se você não quiser executar os testes em paralelo ou quiser controle mais fino sobre o número de threads usadas, pode enviar a flag `--test-threads` e o número de threads que deseja usar ao binário de teste. Veja o exemplo a seguir:

```bash
$ cargo test -- --test-threads=1
```

Definimos o número de threads de teste como `1`, dizendo ao programa para não usar paralelismo. Executar os testes com uma thread levará mais tempo do que executá-los em paralelo, mas os testes não interferirão uns nos outros se compartilharem estado.

### Exibindo a saída das funções

Por padrão, se um teste passa, a biblioteca de testes do Rust captura qualquer coisa impressa na saída padrão. Por exemplo, se chamarmos `println!` em um teste e o teste passar, não veremos a saída de `println!` no terminal; veremos apenas a linha que indica que o teste passou. Se um teste falhar, veremos o que foi impresso na saída padrão junto com o restante da mensagem de falha.

Como exemplo, a Listagem 11-10 tem uma função bobinha que imprime o valor de seu parâmetro e retorna 10, bem como um teste que passa e um que falha.

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

```bash
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

Observe que em lugar nenhum desta saída vemos `I got the value 4`, que é impresso quando o teste que passa executa. Essa saída foi capturada. A saída do teste que falhou, `I got the value 8`, aparece na seção do resumo de testes, que também mostra a causa da falha do teste.

Se quisermos ver valores impressos para testes que passam também, podemos dizer ao Rust para mostrar a saída de testes bem-sucedidos com `--show-output`:

```bash
$ cargo test -- --show-output
```

Quando executamos os testes da Listagem 11-10 novamente com a flag `--show-output`, vemos a seguinte saída:

```bash
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

Executar uma suíte de testes completa às vezes pode levar muito tempo. Se você está trabalhando em código em uma área particular, pode querer executar apenas os testes relacionados a esse código. Você pode escolher quais testes executar passando a `cargo test` o nome ou os nomes do(s) teste(s) que deseja executar como argumento.

Para demonstrar como executar um subconjunto de testes, primeiro criaremos três testes para nossa função `add_two`, como mostrado na Listagem 11-11, e escolheremos quais executar.

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

Se executarmos os testes sem passar argumentos, como vimos antes, todos os testes executarão em paralelo:

```bash
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

Podemos passar o nome de qualquer função de teste a `cargo test` para executar apenas esse teste:

```bash
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

Apenas o teste com o nome `one_hundred` executou; os outros dois testes não corresponderam a esse nome. A saída de teste nos informa que tínhamos mais testes que não executaram exibindo `2 filtered out` no final.

Não podemos especificar os nomes de vários testes desta forma; apenas o primeiro valor dado a `cargo test` será usado. Mas há uma forma de executar vários testes.

#### Filtrando para executar vários testes

Podemos especificar parte de um nome de teste, e qualquer teste cujo nome corresponda a esse valor será executado. Por exemplo, como dois dos nomes dos nossos testes contêm `add`, podemos executar esses dois executando `cargo test add`:

```bash
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

Este comando executou todos os testes com `add` no nome e filtrou o teste chamado `one_hundred`. Observe também que o módulo em que um teste aparece torna-se parte do nome do teste, então podemos executar todos os testes de um módulo filtrando pelo nome do módulo.

### Ignorando testes a menos que especificamente solicitado

Às vezes alguns testes específicos podem ser muito demorados de executar, então você pode querer excluí-los durante a maioria das execuções de `cargo test`. Em vez de listar como argumentos todos os testes que deseja executar, você pode anotar os testes demorados com o atributo `ignore` para excluí-los, como mostrado aqui:

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

Depois de `#[test]`, adicionamos a linha `#[ignore]` ao teste que queremos excluir. Agora, quando executamos nossos testes, `it_works` executa, mas `expensive_test` não:

```bash
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

```bash
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

Ao controlar quais testes executam, você pode garantir que os resultados de `cargo test` retornem rapidamente. Quando estiver em um ponto em que faz sentido verificar os resultados dos testes `ignored` e tiver tempo para esperar pelos resultados, pode executar `cargo test -- --ignored`. Se quiser executar todos os testes, ignorados ou não, pode executar `cargo test -- --include-ignored`.
