---
title: "Aceitando argumentos de linha de comando"
chapter_code: 12-01
slug: aceitando-argumentos-de-linha-de-comando
challenge_day: 15
reading_minutes: 11
---

# Aceitando argumentos de linha de comando

Vamos criar um novo projeto com `cargo new`, como sempre. Chamaremos nosso projeto de `minigrep` para distingui-lo da ferramenta `grep` que você talvez já tenha no seu sistema:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

A primeira tarefa é fazer o `minigrep` aceitar seus dois argumentos de linha de comando: o caminho do arquivo e uma string a ser buscada. Ou seja, queremos conseguir executar nosso programa com `cargo run`, dois hífens para indicar que os argumentos seguintes são para o nosso programa, e não para o `cargo`, uma string a ser buscada e um caminho para o arquivo no qual pesquisar, assim:

```console
$ cargo run -- searchstring example-filename.txt
```

Neste momento, o programa gerado por `cargo new` não consegue processar os argumentos que passamos para ele. Algumas bibliotecas disponíveis em [crates.io](https://crates.io/) podem ajudar na escrita de programas que aceitam argumentos de linha de comando, mas, como você ainda está aprendendo esse conceito, vamos implementar essa capacidade por conta própria.

### Lendo os valores dos argumentos

Para permitir que o `minigrep` leia os valores dos argumentos de linha de comando que passamos para ele, precisaremos da função `std::env::args`, fornecida pela biblioteca padrão do Rust. Essa função retorna um iterator dos argumentos de linha de comando passados ao `minigrep`. Abordaremos iterators em detalhes no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures). Por enquanto, você só precisa saber duas coisas sobre iterators: eles produzem uma série de valores, e podemos chamar o método `collect` em um iterator para transformá-lo em uma coleção, como um vetor, que contém todos os elementos produzidos pelo iterator.

O código da Listagem 12-1 permite que seu programa `minigrep` leia quaisquer argumentos de linha de comando passados a ele e, em seguida, colete esses valores em um vetor.

**Arquivo: src/main.rs**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    dbg!(args);
}
```

<a id="listagem-12-1"></a>

[Listagem 12-1](#listagem-12-1): Coletando os argumentos de linha de comando em um vetor e imprimindo-os

Primeiro, trazemos o módulo `std::env` para o escopo com uma instrução `use`, para que possamos usar sua função `args`. Observe que a função `std::env::args` está aninhada em dois níveis de módulos. Como discutimos no [Capítulo 7](/livro/cap07-00-gerenciando-projetos-crescentes-com-packages-crates-e-modulos), nos casos em que a função desejada está aninhada em mais de um módulo, escolhemos trazer o módulo pai para o escopo em vez da própria função. Fazendo isso, podemos usar facilmente outras funções de `std::env`. Também fica menos ambíguo do que adicionar `use std::env::args` e depois chamar a função apenas com `args`, porque `args` poderia ser facilmente confundida com uma função definida no módulo atual.

> ### A função `args` e Unicode inválido
>
> Observe que `std::env::args` entrará em pânico se algum argumento contiver Unicode inválido. Se seu programa precisar aceitar argumentos que contenham Unicode inválido, use `std::env::args_os` em vez disso. Essa função retorna um iterator que produz valores `OsString`, em vez de valores `String`. Escolhemos usar `std::env::args` aqui por simplicidade, porque valores `OsString` variam de plataforma para plataforma e são mais complexos de trabalhar do que valores `String`.

Na primeira linha de `main`, chamamos `env::args` e imediatamente usamos `collect` para transformar o iterator em um vetor contendo todos os valores produzidos por ele. Podemos usar a função `collect` para criar muitos tipos de coleções, então anotamos explicitamente o tipo de `args` para especificar que queremos um vetor de strings. Embora você raramente precise anotar tipos em Rust, `collect` é uma função que frequentemente exige anotação, porque Rust não consegue inferir qual tipo de coleção você quer.

Por fim, imprimimos o vetor usando a macro de depuração. Vamos tentar executar o código primeiro sem argumentos e depois com dois argumentos:

```console
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/minigrep`
[src/main.rs:5:5] args = [
    "target/debug/minigrep",
]
```

```console
$ cargo run -- needle haystack
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.57s
     Running `target/debug/minigrep needle haystack`
[src/main.rs:5:5] args = [
    "target/debug/minigrep",
    "needle",
    "haystack",
]
```

Observe que o primeiro valor no vetor é `"target/debug/minigrep"`, que é o nome do nosso binário. Isso corresponde ao comportamento da lista de argumentos em C, permitindo que programas usem, durante a execução, o nome pelo qual foram invocados. Muitas vezes é conveniente ter acesso ao nome do programa caso você queira imprimi-lo em mensagens ou mudar o comportamento do programa com base no alias de linha de comando usado para invocá-lo. Mas, para os propósitos deste capítulo, vamos ignorar esse valor e guardar apenas os dois argumentos de que precisamos.

### Salvando os valores dos argumentos em variáveis

O programa agora consegue acessar os valores especificados como argumentos de linha de comando. Precisamos salvar os valores dos dois argumentos em variáveis para poder usá-los durante o restante do programa. Fazemos isso na Listagem 12-2.

**Arquivo: src/main.rs**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Buscando {query}");
    println!("No arquivo {file_path}");
}
```

<a id="listagem-12-2"></a>

[Listagem 12-2](#listagem-12-2): Criando variáveis para guardar o argumento de consulta e o argumento de caminho do arquivo

Como vimos ao imprimir o vetor, o nome do programa ocupa o primeiro valor no vetor, em `args[0]`; portanto, começamos os argumentos no índice 1. O primeiro argumento que o `minigrep` recebe é a string que estamos buscando, então colocamos uma referência ao primeiro argumento na variável `query`. O segundo argumento será o caminho do arquivo, então colocamos uma referência ao segundo argumento na variável `file_path`.

Imprimimos temporariamente os valores dessas variáveis para comprovar que o código está funcionando como pretendemos. Vamos executar este programa novamente com os argumentos `test` e `sample.txt`:

```console
$ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep test sample.txt`
Buscando test
No arquivo sample.txt
```

Ótimo, o programa está funcionando! Os valores dos argumentos de que precisamos estão sendo salvos nas variáveis corretas. Mais adiante, adicionaremos algum tratamento de erros para lidar com certas situações potencialmente problemáticas, como quando o usuário não fornece argumentos; por enquanto, vamos ignorar essa situação e trabalhar na adição da capacidade de ler arquivos.
