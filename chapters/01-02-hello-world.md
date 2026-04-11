---
title: "Hello, world!"
chapter_code: 01-02
slug: hello-world
---

# Hello, World!

Agora que você instalou o Rust, é hora de escrever seu primeiro programa em Rust. É tradicional, ao aprender uma nova linguagem, escrever um pequeno programa que imprime o texto `Hello, world!` na tela, então faremos o mesmo aqui!

> **Nota:** Este livro assume familiaridade básica com a linha de comando. O Rust não faz exigências específicas sobre qual editor ou ferramenta você deve usar ou onde seu código deve ficar, então, se você preferir usar uma IDE em vez da linha de comando, sinta-se à vontade para usar sua IDE favorita. Muitas IDEs já oferecem algum nível de suporte ao Rust; consulte a documentação da IDE para mais detalhes. A equipe do Rust tem se concentrado em oferecer um excelente suporte a IDEs por meio do `rust-analyzer`. Veja o [Apêndice D](#) para mais detalhes.

### Configuração do diretório do projeto

Você começará criando um diretório para armazenar seu código Rust. Para o Rust, não importa onde seu código está localizado, mas para os exercícios e projetos deste livro, sugerimos criar um diretório chamado *projects* no seu diretório pessoal e manter todos os seus projetos lá.

Abra um terminal e digite os seguintes comandos para criar um diretório *projects* e um diretório para o projeto "Hello, world!" dentro dele.

Para Linux, macOS e PowerShell no Windows, digite:

```bash
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Para o CMD do Windows, digite:

```
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Noções básicas de um programa Rust

Em seguida, crie um novo arquivo de código-fonte e chame-o de *main.rs*. Arquivos Rust sempre terminam com a extensão *.rs*. Se você estiver usando mais de uma palavra no nome do arquivo, a convenção é usar um sublinhado para separá-las. Por exemplo, use *hello_world.rs* em vez de *helloworld.rs*.

Agora abra o arquivo *main.rs* que você acabou de criar e insira o código abaixo.

Arquivo: main.rs
```rust
fn main() {
    println!("Hello, world!");
}
```

Salve o arquivo e volte para o terminal no diretório *~/projects/hello_world*. No Linux ou macOS, digite os seguintes comandos para compilar e executar o arquivo:

```bash
$ rustc main.rs
$ ./main
Hello, world!
```

No Windows, use o comando `.\main` em vez de `./main`:

```
> rustc main.rs
> .\main
Hello, world!
```

Independentemente do seu sistema operacional, a string `Hello, world!` deve ser exibida no terminal. Se você não vir essa saída, volte à seção de "Solução de Problemas" da parte de Instalação para obter ajuda.

Se `Hello, world!` foi exibido, parabéns! Você escreveu oficialmente um programa em Rust. Isso faz de você um programador Rust — seja bem-vindo!

### A anatomia de um programa Rust

Vamos revisar esse programa "Hello, world!" em detalhes. Aqui está a primeira parte do quebra-cabeça:

```rust
fn main() {

}
```

Essas linhas definem uma função chamada `main`. A função `main` é especial: ela é sempre o primeiro código executado em todo programa executável em Rust. Aqui, a primeira linha declara uma função chamada `main` que não recebe parâmetros e não retorna nada. Se houvesse parâmetros, eles ficariam dentro dos parênteses `()`.

O corpo da função é delimitado por chaves `{}`. O Rust exige o uso de chaves em todos os corpos de função. É uma boa prática colocar a chave de abertura na mesma linha da declaração da função, com um espaço entre elas.

> **Nota:** Se você quiser manter um estilo padrão em projetos Rust, pode usar uma ferramenta de formatação automática chamada `rustfmt`. A equipe do Rust inclui essa ferramenta na distribuição padrão, assim como o `rustc`, então ela provavelmente já está instalada no seu computador.

O corpo da função `main` contém o seguinte código:

```rust
println!("Hello, world!");
```

Essa linha faz todo o trabalho deste pequeno programa: ela imprime texto na tela. Há três detalhes importantes a observar aqui.

Primeiro, `println!` chama uma macro do Rust. Se fosse uma função, seria escrita como `println`, sem o ponto de exclamação. Macros são uma forma de escrever código que gera código e estende a sintaxe do Rust.

Segundo, temos a string `"Hello, world!"`, que é passada como argumento para `println!` e exibida na tela.

Terceiro, a linha termina com um ponto e vírgula (`;`), indicando o fim da expressão. A maioria das linhas em Rust termina com um ponto e vírgula.

### Compilação e execução

Você acabou de executar um programa recém-criado, então vamos examinar cada etapa desse processo.

Antes de executar um programa Rust, é necessário compilá-lo usando o compilador Rust com o comando `rustc`, passando o nome do arquivo-fonte:

```bash
$ rustc main.rs
```

Se você tem experiência com C ou C++, perceberá que isso é semelhante ao uso do `gcc` ou `clang`. Após a compilação bem-sucedida, o Rust gera um executável binário.

No Linux, macOS e PowerShell no Windows, você pode ver o executável usando o comando `ls`:

```bash
$ ls
main  main.rs
```

No CMD do Windows, você verá:

```
> dir /B
main.exe
main.pdb
main.rs
```

Isso mostra o arquivo-fonte (*.rs*), o executável (*main* ou *main.exe*) e, no Windows, um arquivo de depuração (*.pdb*). Para executar o programa:

```bash
$ ./main   # ou .\main no Windows
```

Se *main.rs* for o programa "Hello, world!", essa linha exibirá `Hello, world!` no terminal.

Diferente de linguagens dinâmicas como Ruby, Python ou JavaScript, o Rust é uma linguagem compilada antecipadamente. Isso significa que você pode compilar um programa e entregar o executável para outra pessoa executar, mesmo que ela não tenha o Rust instalado.

Compilar apenas com `rustc` é suficiente para programas simples, mas conforme seu projeto cresce, você vai querer uma forma melhor de gerenciar opções e dependências. A seguir, você conhecerá o Cargo, a ferramenta que ajuda a escrever programas Rust do mundo real.