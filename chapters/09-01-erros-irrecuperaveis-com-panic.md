---
title: "Erros irrecuperáveis com `panic!`"
chapter_code: 09-01
slug: erros-irrecuperaveis-com-panic
---

# Erros Irrecuperáveis com `panic!`

Às vezes coisas ruins acontecem no seu código, e não há nada que você possa fazer a respeito. Nesses casos, o Rust tem a macro `panic!`. Há duas formas de causar um pânico na prática: tomando uma ação que faz nosso código entrar em pânico (como acessar um array além do fim) ou chamando explicitamente a macro `panic!`. Em ambos os casos, causamos um pânico em nosso programa. Por padrão, esses pânicos imprimirão uma mensagem de falha, farão unwind, limparão a stack e encerrarão. Por meio de uma variável de ambiente, você também pode fazer o Rust exibir a call stack quando um pânico ocorre para facilitar rastrear a origem do pânico.

> ### Fazendo unwind da stack ou abortando em resposta a um pânico
>
> Por padrão, quando um pânico ocorre, o programa começa a fazer _unwind_, o que significa que o Rust percorre a stack de volta e limpa os dados de cada função que encontra. Porém, percorrer e limpar é muito trabalho. O Rust, portanto, permite escolher a alternativa de _abortar_ imediatamente, o que encerra o programa sem limpar.
>
> A memória que o programa estava usando precisará então ser limpa pelo sistema operacional. Se no seu projeto você precisa tornar o binário resultante o menor possível, pode mudar de unwind para abort em um pânico adicionando `panic = 'abort'` às seções `[profile]` apropriadas no seu arquivo _Cargo.toml_. Por exemplo, se quiser abortar em pânico no modo release, adicione isto:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Vamos tentar chamar `panic!` em um programa simples:

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
fn main() {
    panic!("crash and burn");
}
```

Quando você executar o programa, verá algo assim:

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`

thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

A chamada a `panic!` causa a mensagem de erro contida nas duas últimas linhas. A primeira linha mostra nossa mensagem de pânico e o local no código-fonte onde o pânico ocorreu: _src/main.rs:2:5_ indica que é a segunda linha, quinto caractere do nosso arquivo _src/main.rs_.

Neste caso, a linha indicada faz parte do nosso código, e se formos a essa linha, veremos a chamada à macro `panic!`. Em outros casos, a chamada a `panic!` pode estar em código que nosso código chama, e o nome do arquivo e o número da linha reportados pela mensagem de erro serão de código de outra pessoa onde a macro `panic!` é chamada, não a linha do nosso código que eventualmente levou à chamada a `panic!`.

Podemos usar o backtrace das funções das quais a chamada a `panic!` veio para descobrir a parte do nosso código que está causando o problema. Para entender como usar um backtrace de `panic!`, vamos olhar outro exemplo e ver como é quando uma chamada a `panic!` vem de uma biblioteca por causa de um bug no nosso código em vez de nosso código chamar a macro diretamente. A Listagem 9-1 tem código que tenta acessar um índice em um vetor além do intervalo de índices válidos.

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

[Listagem 9-1](#listagem-9-1): Tentando acessar um elemento além do fim de um vetor, o que causará uma chamada a `panic!`

Aqui, estamos tentando acessar o 100º elemento do nosso vetor (que está no índice 99 porque a indexação começa em zero), mas o vetor tem apenas três elementos. Nesta situação, o Rust entrará em pânico. Usar `[]` deveria retornar um elemento, mas se você passar um índice inválido, não há elemento que o Rust pudesse retornar aqui que seria correto.

Em C, tentar ler além do fim de uma estrutura de dados é comportamento indefinido. Você pode obter o que estiver na localização na memória que corresponderia a esse elemento na estrutura de dados, mesmo que a memória não pertença a essa estrutura. Isso é chamado de _buffer overread_ e pode levar a vulnerabilidades de segurança se um atacante conseguir manipular o índice de forma a ler dados que não deveria ter permissão de ler, armazenados após a estrutura de dados.

Para proteger seu programa desse tipo de vulnerabilidade, se você tentar ler um elemento em um índice que não existe, o Rust interromperá a execução e se recusará a continuar. Vamos tentar e ver:

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`

thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Este erro aponta para a linha 4 do nosso _main.rs_, onde tentamos acessar o índice 99 do vetor em `v`.

A linha `note:` nos diz que podemos definir a variável de ambiente `RUST_BACKTRACE` para obter um backtrace de exatamente o que aconteceu para causar o erro. Um _backtrace_ é uma lista de todas as funções que foram chamadas para chegar a este ponto. Backtraces em Rust funcionam como em outras linguagens: a chave para ler o backtrace é começar do topo e ler até ver arquivos que você escreveu. Esse é o ponto onde o problema se originou. As linhas acima desse ponto são código que seu código chamou; as linhas abaixo são código que chamou seu código. Essas linhas antes e depois podem incluir código principal do Rust, código da biblioteca padrão ou crates que você está usando. Vamos tentar obter um backtrace definindo a variável de ambiente `RUST_BACKTRACE` para qualquer valor exceto `0`. A Listagem 9-2 mostra saída semelhante ao que você verá.

[Listagem 9-2](#listagem-9-2): O backtrace gerado por uma chamada a `panic!` exibido quando a variável de ambiente `RUST_BACKTRACE` está definida

```bash
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Isso é muita saída! A saída exata que você vê pode ser diferente dependendo do seu sistema operacional e da versão do Rust. Para obter backtraces com essas informações, símbolos de depuração devem estar habilitados. Símbolos de depuração estão habilitados por padrão ao usar `cargo build` ou `cargo run` sem a flag `--release`, como fizemos aqui.

Na saída da Listagem 9-2, a linha 6 do backtrace aponta para a linha do nosso projeto que está causando o problema: linha 4 de _src/main.rs_. Se não quisermos que nosso programa entre em pânico, devemos começar nossa investigação no local apontado pela primeira linha que menciona um arquivo que escrevemos. Na Listagem 9-1, onde escrevemos deliberadamente código que entraria em pânico, a forma de corrigir o pânico é não solicitar um elemento além do intervalo dos índices do vetor. Quando seu código entrar em pânico no futuro, você precisará descobrir qual ação o código está tomando com quais valores para causar o pânico e o que o código deveria fazer em vez disso.

Voltaremos a `panic!` e quando devemos e não devemos usar `panic!` para lidar com condições de erro na seção Panic ou não panic mais adiante neste capítulo. Em seguida, veremos como recuperar de um erro usando `Result`.
