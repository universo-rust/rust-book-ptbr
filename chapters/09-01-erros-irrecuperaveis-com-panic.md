---
title: "Erros irrecuperáveis com `panic!`"
chapter_code: 09-01
slug: erros-irrecuperaveis-com-panic
challenge_day: 11
reading_minutes: 7
---

# Erros irrecuperáveis com `panic!`

Às vezes algo dá errado no código e não há o que fazer. Nesses casos, o Rust oferece a macro `panic!`. Na prática, há duas formas de provocar um pânico: tomar uma ação que faz o programa entrar em pânico (como acessar um array além do fim) ou chamar `panic!` explicitamente. Em ambos os casos, o programa entra em pânico. Por padrão, isso imprime uma mensagem de falha, faz _unwind_, limpa a stack e encerra. Com uma variável de ambiente, você também pode pedir ao Rust que exiba a call stack quando o pânico ocorrer — o que facilita localizar a origem.

> ### Fazendo unwind da stack ou abortando em resposta a um pânico
>
> Por padrão, quando ocorre um pânico, o programa começa a fazer _unwind_: o Rust percorre a stack de volta e limpa os dados de cada função encontrada. Percorrer e limpar, porém, dá trabalho. Por isso o Rust permite _abortar_ imediatamente — encerrar o programa sem limpar.
>
> A memória usada pelo programa passa a ser liberada pelo sistema operacional. Se você precisa de um binário o menor possível, pode trocar unwind por abort no pânico adicionando `panic = 'abort'` nas seções `[profile]` adequadas do _Cargo.toml_. Por exemplo, para abortar em pânico no modo release:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Vamos chamar `panic!` num programa simples:

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
fn main() {
    panic!("crash and burn");
}
```

Ao executar, você verá algo assim:

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`

thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

A chamada a `panic!` produz a mensagem nas duas últimas linhas. A primeira traz a mensagem de pânico e o ponto no código-fonte: _src/main.rs:2:5_ indica a segunda linha, quinto caractere do arquivo _src/main.rs_.

Neste caso, a linha indicada é do nosso código — ao ir até ela, vemos a chamada a `panic!`. Em outros casos, o `panic!` pode estar em código que o nosso chama; o arquivo e a linha reportados serão de outra pessoa, onde a macro é invocada, e não da linha do nosso código que levou ao pânico.

O backtrace das funções envolvidas ajuda a achar a parte do nosso código que causou o problema. Para entender como usá-lo, vejamos outro exemplo: um `panic!` vindo de uma biblioteca por causa de um bug no nosso código, em vez de uma chamada direta à macro. A Listagem 9-1 tenta acessar um índice de vetor fora do intervalo válido.

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<a id="listagem-9-1"></a>

[Listagem 9-1](#listagem-9-1): Tentando acessar um elemento além do fim de um vetor, o que provoca uma chamada a `panic!`

Aqui tentamos acessar o 100º elemento do vetor (índice 99, porque a indexação começa em zero), mas o vetor só tem três. Nessa situação, o Rust entra em pânico. `[]` deveria devolver um elemento — mas com índice inválido não há retorno correto possível.

Em C, ler além do fim de uma estrutura de dados é comportamento indefinido. Você pode obter o que estiver na memória na posição correspondente ao elemento, mesmo que essa memória não pertença à estrutura. Isso se chama _buffer overread_ e pode gerar vulnerabilidades se um atacante manipular o índice para ler dados que não deveria, armazenados depois da estrutura.

Para proteger o programa, o Rust interrompe a execução ao tentar ler um índice inexistente. Vejamos na prática:

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`

thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

O erro aponta para a linha 4 do _main.rs_, onde acessamos o índice 99 de `v`.

A linha `note:` informa que podemos definir `RUST_BACKTRACE` para obter um backtrace do que levou ao erro. Um _backtrace_ é a lista de funções chamadas até aquele ponto. Em Rust, funciona como em outras linguagens: comece pelo topo e leia até encontrar arquivos que você escreveu — aí está a origem do problema. Acima disso, código que o seu chamou; abaixo, código que chamou o seu. Essas linhas podem incluir o núcleo do Rust, a biblioteca padrão ou crates que você usa. Vamos definir `RUST_BACKTRACE` com qualquer valor diferente de `0`. A Listagem 9-2 mostra saída parecida com a que você verá.

<a id="listagem-9-2"></a>

[Listagem 9-2](#listagem-9-2): Backtrace gerado por `panic!` com a variável de ambiente `RUST_BACKTRACE` definida

```console
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

É bastante saída! O resultado exato pode variar conforme o sistema operacional e a versão do Rust. Para backtraces com esse nível de detalhe, os símbolos de depuração precisam estar habilitados — o que é o padrão em `cargo build` ou `cargo run` sem `--release`, como aqui.

Na Listagem 9-2, a linha 6 do backtrace aponta para o problema no nosso projeto: linha 4 de _src/main.rs_. Se não queremos pânico, a investigação começa na primeira linha que menciona um arquivo nosso. Na Listagem 9-1, onde provocamos o pânico de propósito, a correção é simples: não pedir um elemento fora do intervalo do vetor. Quando o código entrar em pânico no futuro, será preciso descobrir qual ação, com quais valores, causou o pânico — e o que deveria acontecer em vez disso.

Voltaremos a `panic!` — e a quando usar ou não `panic!` para tratar erros — na seção [Panic ou Não Panic](/livro/cap09-03-panic-ou-nao-panic), mais adiante neste capítulo. Em seguida, veremos como se recuperar de um erro com `Result`.
