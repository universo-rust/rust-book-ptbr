---
title: "Aceitando argumentos de linha de comando"
chapter_code: 12-01
slug: aceitando-argumentos-de-linha-de-comando
---

# Aceitando Argumentos de Linha de Comando

Vamos criar um novo projeto com, como sempre, `cargo new`. Chamaremos nosso projeto de `minigrep` para distingui-lo da ferramenta `grep` que você talvez já tenha no sistema:

```bash
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

A primeira tarefa é fazer `minigrep` aceitar seus dois argumentos de linha de comando: o caminho do arquivo e uma string para buscar. Ou seja, queremos poder executar nosso programa com `cargo run`, dois hífens para indicar que os argumentos seguintes são do nosso programa e não do `cargo`, uma string para buscar e um caminho para o arquivo a ser pesquisado, assim:

```bash
$ cargo run -- searchstring example-filename.txt
```

No momento, o programa gerado por `cargo new` não consegue processar os argumentos que passamos a ele. Algumas bibliotecas existentes em [crates.io](https://crates.io/) podem ajudar a escrever um programa que aceita argumentos de linha de comando, mas como você está aprendendo este conceito, vamos implementar essa capacidade nós mesmos.

### Lendo os valores dos argumentos

Para permitir que `minigrep` leia os valores dos argumentos de linha de comando que passamos a ele, precisaremos da função `std::env::args` fornecida na biblioteca padrão do Rust. Esta função retorna um iterator dos argumentos de linha de comando passados a `minigrep`. Cobriremos iterators por completo no Capítulo 13. Por enquanto, você só precisa saber dois detalhes sobre iterators: iterators produzem uma série de valores, e podemos chamar o método `collect` em um iterator para transformá-lo em uma coleção, como um vetor, que contém todos os elementos que o iterator produz.

O código da Listagem 12-1 permite que seu programa `minigrep` leia quaisquer argumentos de linha de comando passados a ele e então colete os valores em um vetor.

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

Primeiro, trazemos o módulo `std::env` para o escopo com uma instrução `use` para podermos usar sua função `args`. Observe que a função `std::env::args` está aninhada em dois níveis de módulos. Como discutimos no Capítulo 7, em casos em que a função desejada está aninhada em mais de um módulo, escolhemos trazer o módulo pai para o escopo em vez da função. Ao fazer isso, podemos usar facilmente outras funções de `std::env`. Também é menos ambíguo do que adicionar `use std::env::args` e então chamar a função apenas com `args`, porque `args` poderia facilmente ser confundida com uma função definida no módulo atual.

> ### A função `args` e Unicode inválido
>
> Observe que `std::env::args` entrará em pânico se qualquer argumento contiver Unicode inválido. Se seu programa precisar aceitar argumentos contendo Unicode inválido, use `std::env::args_os` em vez disso. Essa função retorna um iterator que produz valores `OsString` em vez de valores `String`. Escolhemos usar `std::env::args` aqui por simplicidade, porque valores `OsString` diferem por plataforma e são mais complexos de trabalhar do que valores `String`.

Na primeira linha de `main`, chamamos `env::args` e imediatamente usamos `collect` para transformar o iterator em um vetor contendo todos os valores produzidos pelo iterator. Podemos usar a função `collect` para criar muitos tipos de coleções, então anotamos explicitamente o tipo de `args` para especificar que queremos um vetor de strings. Embora você raramente precise anotar tipos em Rust, `collect` é uma função em que você frequentemente precisa anotar, porque o Rust não consegue inferir o tipo de coleção que você quer.

Por fim, imprimimos o vetor usando a macro de debug. Vamos tentar executar o código primeiro sem argumentos e depois com dois argumentos:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/minigrep`
[src/main.rs:5:5] args = [
    "target/debug/minigrep",
]
```

```bash
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

Observe que o primeiro valor no vetor é `"target/debug/minigrep"`, que é o nome do nosso binário. Isso corresponde ao comportamento da lista de argumentos em C, permitindo que programas usem o nome pelo qual foram invocados em sua execução. Muitas vezes é conveniente ter acesso ao nome do programa caso queira imprimi-lo em mensagens ou mudar o comportamento do programa com base em qual alias de linha de comando foi usado para invocar o programa. Mas, para os propósitos deste capítulo, ignoraremos isso e guardaremos apenas os dois argumentos que precisamos.

### Salvando os valores dos argumentos em variáveis

O programa já consegue acessar os valores especificados como argumentos de linha de comando. Agora precisamos salvar os valores dos dois argumentos em variáveis para podermos usá-los no restante do programa. Fazemos isso na Listagem 12-2.

**Arquivo: src/main.rs**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Searching for {query}");
    println!("In file {file_path}");
}
```

<a id="listagem-12-2"></a>

[Listagem 12-2](#listagem-12-2): Criando variáveis para guardar o argumento de consulta e o argumento de caminho do arquivo

Como vimos ao imprimir o vetor, o nome do programa ocupa o primeiro valor no vetor em `args[0]`, então começamos os argumentos no índice 1. O primeiro argumento que `minigrep` recebe é a string que estamos buscando, então colocamos uma referência ao primeiro argumento na variável `query`. O segundo argumento será o caminho do arquivo, então colocamos uma referência ao segundo argumento na variável `file_path`.

Imprimimos temporariamente os valores dessas variáveis para provar que o código está funcionando como pretendemos. Vamos executar este programa novamente com os argumentos `test` e `sample.txt`:

```bash
$ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

Ótimo, o programa está funcionando! Os valores dos argumentos que precisamos estão sendo salvos nas variáveis certas. Mais tarde adicionaremos algum tratamento de erros para lidar com certas situações potencialmente errôneas, como quando o usuário não fornece argumentos; por enquanto, ignoraremos essa situação e trabalharemos em adicionar capacidade de leitura de arquivo.
