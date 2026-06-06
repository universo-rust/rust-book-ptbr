---
title: "Workspaces do Cargo"
chapter_code: 14-03
slug: workspaces-do-cargo
challenge_day: 18
reading_minutes: 8
---

# Workspaces do Cargo

No Capítulo 12, construímos um pacote que incluía um crate binário e um crate de biblioteca. Conforme seu projeto se desenvolve, você pode descobrir que o crate de biblioteca continua crescendo e quer dividir seu pacote ainda mais em vários crates de biblioteca. O Cargo oferece um recurso chamado _workspaces_ que pode ajudar a gerenciar vários pacotes relacionados desenvolvidos em conjunto.

## Criando um workspace

Um _workspace_ é um conjunto de pacotes que compartilham o mesmo _Cargo.lock_ e diretório de saída. Vamos fazer um projeto usando um workspace — usaremos código trivial para podermos nos concentrar na estrutura do workspace. Há várias formas de estruturar um workspace; mostraremos apenas uma forma comum. Teremos um workspace contendo um binário e duas bibliotecas. O binário, que fornecerá a funcionalidade principal, dependerá das duas bibliotecas. Uma biblioteca fornecerá uma função `add_one` e a outra biblioteca uma função `add_two`. Esses três crates farão parte do mesmo workspace. Começaremos criando um novo diretório para o workspace:

```console
$ mkdir add
$ cd add
```

Em seguida, no diretório _add_, criamos o arquivo _Cargo.toml_ que configurará o workspace inteiro. Este arquivo não terá uma seção `[package]`. Em vez disso, começará com uma seção `[workspace]` que nos permitirá adicionar membros ao workspace. Também definimos explicitamente o algoritmo _resolver_ mais recente do Cargo em nosso workspace definindo o valor `resolver` como `"3"`:

**Arquivo: Cargo.toml**

```toml
[workspace]
resolver = "3"
```

Em seguida, criaremos o crate binário `adder` executando `cargo new` dentro do diretório _add_:

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Executar `cargo new` dentro de um workspace também adiciona automaticamente o pacote recém-criado à chave `members` na definição `[workspace]` no _Cargo.toml_ do workspace, assim:

```toml
[workspace]
resolver = "3"
members = ["adder"]
```

Neste ponto, podemos compilar o workspace executando `cargo build`. Os arquivos no seu diretório _add_ devem se parecer com isto:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

O workspace tem um diretório _target_ no nível superior para onde os artefatos compilados serão colocados; o pacote `adder` não tem seu próprio diretório _target_. Mesmo que executássemos `cargo build` de dentro do diretório _adder_, os artefatos compilados ainda iriam para _add/target_ em vez de _add/adder/target_. O Cargo estrutura o diretório _target_ em um workspace assim porque os crates em um workspace devem depender uns dos outros. Se cada crate tivesse seu próprio diretório _target_, cada crate teria que recompilar cada um dos outros crates no workspace para colocar os artefatos em seu próprio diretório _target_. Ao compartilhar um diretório _target_, os crates podem evitar recompilações desnecessárias.

## Criando o segundo pacote no workspace

Em seguida, vamos criar outro pacote membro no workspace e chamá-lo `add_one`. Gere um novo crate de biblioteca chamado `add_one`:

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

O _Cargo.toml_ de nível superior agora incluirá o caminho _add_one_ na lista `members`:

**Arquivo: Cargo.toml**

```toml
[workspace]
resolver = "3"
members = ["adder", "add_one"]
```

Seu diretório _add_ agora deve ter estes diretórios e arquivos:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

No arquivo _add_one/src/lib.rs_, vamos adicionar uma função `add_one`:

**Arquivo: add_one/src/lib.rs**

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Agora podemos fazer o pacote `adder` com nosso binário depender do pacote `add_one` que tem nossa biblioteca. Primeiro, precisaremos adicionar uma dependência de caminho em `add_one` ao _adder/Cargo.toml_.

**Arquivo: adder/Cargo.toml**

```toml
add_one = { path = "../add_one" }
```

O Cargo não assume que crates em um workspace dependerão uns dos outros, então precisamos ser explícitos sobre as relações de dependência.

Em seguida, vamos usar a função `add_one` (do crate `add_one`) no crate `adder`. Abra o arquivo _adder/src/main.rs_ e altere a função `main` para chamar a função `add_one`, como na Listagem 14-7.

**Arquivo: adder/src/main.rs**

```rust
fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
}
```

<a id="listagem-14-7"></a>

[Listagem 14-7](#listagem-14-7): Usando o crate de biblioteca `add_one` a partir do crate `adder`

Vamos compilar o workspace executando `cargo build` no diretório _add_ de nível superior!

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Para executar o crate binário a partir do diretório _add_, podemos especificar qual pacote no workspace queremos executar usando o argumento `-p` e o nome do pacote com `cargo run`:

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Isso executa o código em _adder/src/main.rs_, que depende do crate `add_one`.

## Dependendo de um pacote externo

Observe que o workspace tem apenas um arquivo _Cargo.lock_ no nível superior, em vez de ter um _Cargo.lock_ no diretório de cada crate. Isso garante que todos os crates usem a mesma versão de todas as dependências. Se adicionarmos o pacote `rand` aos arquivos _adder/Cargo.toml_ e _add_one/Cargo.toml_, o Cargo resolverá ambos para uma versão de `rand` e registrará isso no único _Cargo.lock_. Fazer todos os crates no workspace usarem as mesmas dependências significa que os crates estarão sempre compatíveis uns com os outros. Vamos adicionar o crate `rand` à seção `[dependencies]` no arquivo _add_one/Cargo.toml_ para podermos usar o crate `rand` no crate `add_one`:

**Arquivo: add_one/Cargo.toml**

```toml
rand = "0.8.5"
```

Agora podemos adicionar `use rand;` ao arquivo _add_one/src/lib.rs_, e compilar o workspace inteiro executando `cargo build` no diretório _add_ trará e compilará o crate `rand`. Obteremos um aviso porque não estamos referenciando o `rand` que trouxemos para o escopo:

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

O _Cargo.lock_ de nível superior agora contém informação sobre a dependência de `add_one` em `rand`. No entanto, mesmo que `rand` seja usado em algum lugar do workspace, não podemos usá-lo em outros crates no workspace a menos que adicionemos `rand` aos arquivos _Cargo.toml_ deles também. Por exemplo, se adicionarmos `use rand;` ao arquivo _adder/src/main.rs_ para o pacote `adder`, obteremos um erro:

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Para corrigir isso, edite o arquivo _Cargo.toml_ do pacote `adder` e indique que `rand` é uma dependência dele também. Compilar o pacote `adder` adicionará `rand` à lista de dependências de `adder` no _Cargo.lock_, mas nenhuma cópia adicional de `rand` será baixada. O Cargo garantirá que todo crate em todo pacote no workspace que usar o pacote `rand` usará a mesma versão, desde que especifiquem versões compatíveis de `rand`, economizando espaço e garantindo que os crates no workspace serão compatíveis uns com os outros.

Se crates no workspace especificarem versões incompatíveis da mesma dependência, o Cargo resolverá cada uma delas, mas ainda tentará resolver o menor número possível de versões.

## Adicionando um teste a um workspace

Para outra melhoria, vamos adicionar um teste da função `add_one::add_one` dentro do crate `add_one`:

**Arquivo: add_one/src/lib.rs**

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

Agora execute `cargo test` no diretório _add_ de nível superior. Executar `cargo test` em um workspace estruturado assim executará os testes de todos os crates no workspace:

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

A primeira seção da saída mostra que o teste `it_works` no crate `add_one` passou. A próxima seção mostra que zero testes foram encontrados no crate `adder`, e então a última seção mostra que zero documentation tests foram encontrados no crate `add_one`.

Também podemos executar testes para um crate específico em um workspace a partir do diretório de nível superior usando a flag `-p` e especificando o nome do crate que queremos testar:

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Esta saída mostra que `cargo test` executou apenas os testes do crate `add_one` e não executou os testes do crate `adder`.

Se você publicar os crates no workspace em [crates.io](https://crates.io/), cada crate no workspace precisará ser publicado separadamente. Como `cargo test`, podemos publicar um crate específico em nosso workspace usando a flag `-p` e especificando o nome do crate que queremos publicar.

Para prática adicional, adicione um crate `add_two` a este workspace de forma semelhante ao crate `add_one`!

Conforme seu projeto cresce, considere usar um workspace: ele permite trabalhar com componentes menores e mais fáceis de entender do que um grande bloco de código. Além disso, manter os crates em um workspace pode facilitar a coordenação entre crates se eles são frequentemente alterados ao mesmo tempo.
