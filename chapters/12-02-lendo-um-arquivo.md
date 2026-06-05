---
title: "Lendo um arquivo"
chapter_code: 12-02
slug: lendo-um-arquivo
---

# Lendo um arquivo

Agora adicionaremos a funcionalidade para ler o arquivo especificado no argumento `file_path`. Primeiro, precisamos de um arquivo de exemplo para testar: usaremos um arquivo com uma pequena quantidade de texto distribuída em várias linhas, com algumas palavras repetidas. A Listagem 12-3 traz um poema de Emily Dickinson que servirá muito bem! Crie um arquivo chamado _poema.txt_ no nível raiz do seu projeto e insira o poema “Não sou ninguém! Quem é você?”

**Arquivo: poema.txt**

```text
Não sou ninguém! Quem é você?
Você também não é ninguém?
Então somos um par — não conte!
Eles nos baniriam, sabe.

Que tedioso ser alguém!
Que exposto, como um sapo
Dizer o seu nome o dia inteiro
A um pântano admirador!
```

<a id="listagem-12-3"></a>

[Listagem 12-3](#listagem-12-3): Um poema de Emily Dickinson é um bom caso de teste

Com o texto no lugar, edite _src/main.rs_ e adicione o código para ler o arquivo, como mostrado na Listagem 12-4.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Buscando {query}");
    println!("No arquivo {file_path}");

    let contents = fs::read_to_string(file_path)
        .expect("deveria ter conseguido ler o arquivo");

    println!("Com o texto:\n{contents}");
}
```

<a id="listagem-12-4"></a>

[Listagem 12-4](#listagem-12-4): Lendo o conteúdo do arquivo especificado pelo segundo argumento

Primeiro, trazemos para o escopo uma parte relevante da biblioteca padrão com uma instrução `use`: precisamos de `std::fs` para lidar com arquivos.

Em `main`, a nova instrução `fs::read_to_string` recebe o `file_path`, abre esse arquivo e retorna um valor do tipo `std::io::Result<String>` que contém o conteúdo do arquivo.

Depois disso, adicionamos novamente uma instrução temporária `println!` que imprime o valor de `contents` depois que o arquivo é lido, para que possamos verificar que o programa está funcionando até aqui.

Vamos executar este código com qualquer string como primeiro argumento de linha de comando (porque ainda não implementamos a parte de busca) e o arquivo _poema.txt_ como segundo argumento:

```console
$ cargo run -- o poema.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep o poema.txt`
Buscando o
No arquivo poema.txt
Com o texto:
Não sou ninguém! Quem é você?
Você também não é ninguém?
Então somos um par — não conte!
Eles nos baniriam, sabe.

Que tedioso ser alguém!
Que exposto, como um sapo
Dizer o seu nome o dia inteiro
A um pântano admirador!
```

Ótimo! O código leu e depois imprimiu o conteúdo do arquivo. Mas ele tem algumas falhas. Neste momento, a função `main` tem várias responsabilidades: em geral, funções ficam mais claras e mais fáceis de manter quando cada uma é responsável por apenas uma ideia. O outro problema é que não estamos tratando erros tão bem quanto poderíamos. O programa ainda é pequeno, então essas falhas não são um grande problema, mas, à medida que o programa crescer, ficará mais difícil corrigi-las de forma limpa. É uma boa prática começar a refatorar cedo durante o desenvolvimento de um programa, porque é muito mais fácil refatorar pequenas quantidades de código. Faremos isso a seguir.
