---
title: "Programando um jogo de adivinhação"
chapter_code: 02-00
slug: programando-um-jogo-de-adivinhacao
---

# Programando um Jogo de Adivinhação

Vamos mergulhar no Rust trabalhando em um projeto prático juntos! Este capítulo apresenta alguns conceitos comuns do Rust mostrando como usá-los em um programa real. Você aprenderá sobre `let`, `match`, métodos, funções associadas, crates externos e mais! Nos capítulos seguintes, exploraremos essas ideias em mais detalhes. Neste capítulo, você apenas praticará os fundamentos.

Implementaremos um problema clássico de programação para iniciantes: um jogo de adivinhação. Veja como funciona: O programa gerará um número inteiro aleatório entre 1 e 100. Em seguida, solicitará que o jogador digite um palpite. Após um palpite ser digitado, o programa indicará se o palpite é muito baixo ou muito alto. Se o palpite estiver correto, o jogo imprimirá uma mensagem de parabéns e encerrará.

## Configurando um novo projeto

Para configurar um novo projeto, vá para o diretório *projects* que você criou no Capítulo 1 e crie um novo projeto usando Cargo, assim:

```bash
$ cargo new guessing_game
$ cd guessing_game
```

O primeiro comando, `cargo new`, recebe o nome do projeto (`guessing_game`) como primeiro argumento. O segundo comando muda para o diretório do novo projeto.

Observe o arquivo *Cargo.toml* gerado:

Arquivo: Cargo.toml
```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"

[dependencies]
```

Como você viu no Capítulo 1, `cargo new` gera um programa "Hello, world!" para você. Confira o arquivo *src/main.rs*:

Arquivo: src/main.rs
```rust
fn main() {
    println!("Hello, world!");
}
```

Agora vamos compilar este programa "Hello, world!" e executá-lo no mesmo passo usando o comando `cargo run`:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/guessing_game`
Hello, world!
```

O comando `run` é útil quando você precisa iterar rapidamente em um projeto, como faremos neste jogo, testando rapidamente cada iteração antes de passar para a próxima.

Reabra o arquivo *src/main.rs*. Você escreverá todo o código neste arquivo.

## Processando um palpite

A primeira parte do programa de jogo de adivinhação pedirá entrada do usuário, processará essa entrada e verificará se a entrada está no formato esperado. Para começar, permitiremos que o jogador digite um palpite. Digite o código da Listagem 2-1 em *src/main.rs*.

Arquivo: src/main.rs
```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
      .read_line(&mut guess)
      .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```
[Listagem 2-1](#listagem-2-1): Código que recebe um palpite do usuário e o imprime.

Este código contém muita informação, então vamos analisá-lo linha por linha. Para obter a entrada do usuário e imprimir o resultado como saída, precisamos incluir a biblioteca de entrada/saída `io` no escopo. A biblioteca `io` vem da biblioteca padrão, conhecida como `std`:

```rust
use std::io;
```

Por padrão, o Rust possui um conjunto de itens definidos na biblioteca padrão que são incorporados ao escopo de todos os programas. Este conjunto é chamado de *prelude*, e você pode ver tudo sobre ele na [documentação da biblioteca padrão](https://doc.rust-lang.org/std/prelude/index.html).

Se um tipo que você deseja usar não estiver no *prelude*, você tem que trazer esse tipo para o escopo explicitamente com uma declaração `use`. Usar a biblioteca `std::io` fornece vários recursos úteis, incluindo a capacidade de aceitar entrada do usuário.

Como você viu no Capítulo 1, a função `main` é o ponto de entrada no programa:

```rust
fn main() {
```

A sintaxe `fn` declara uma nova função; os parênteses, `()`, indicam que não há parâmetros; e a chave, `{`, inicia o corpo da função.

Como você também aprendeu no Capítulo 1, `println!` é uma macro que imprime uma string na tela:

```rust
println!("Guess the number!");

println!("Please input your guess.");
```

Este código imprime uma mensagem informando qual é o jogo e solicitando a entrada do usuário.

### Armazenando valores com variáveis

Em seguida, criaremos uma variável para armazenar a entrada do usuário, assim:

```rust
let mut guess = String::new();
```

Agora o programa está ficando interessante! Há muito acontecendo nesta pequena linha. Usamos a declaração `let` para criar a variável. Aqui está outro exemplo:

```rust
let apples = 5;
```

Esta linha cria uma nova variável chamada `apples` e a atribui o valor 5. Em Rust, variáveis são imutáveis por padrão, o que significa que uma vez que damos à variável um valor, o valor não mudará. Discutiremos este conceito em detalhes na seção [Variáveis e Mutabilidade](#) no Capítulo 3. Para tornar uma variável mutável, adicionamos `mut` antes do nome da variável:

```rust
let apples = 5; // imutável
let mut bananas = 5; // mutável
```

> **Nota:** A sintaxe `//` inicia um comentário que continua até o final da linha. Rust ignora tudo o que estiver dentro de comentários. Discutiremos comentários em mais detalhes no Capítulo 3.

Voltando ao programa do jogo de adivinhação, agora você sabe que `let mut guess` cria uma variável mutável chamada `guess`. O sinal de igual (`=`) indica ao Rust que queremos atribuir um valor à variável. À direita do sinal de igual está o valor atribuído a `guess`, que é o resultado da chamada de `String::new`, uma função que retorna uma nova instância de `String`. `String` é um tipo de string fornecido pela biblioteca padrão, que representa um texto codificado em UTF-8 e que pode aumentar de tamanho conforme mais texto é adicionado.

A sintaxe `::` na linha `::new` indica que `new` é uma função associada do tipo `String`. Uma função associada é uma função que é implementada em um tipo, neste caso `String`. Esta função `new` cria uma nova string vazia. Você encontrará uma função `new` em muitos tipos porque é um nome comum para uma função que cria um novo valor de algum tipo.

Na íntegra, a linha `let mut guess = String::new();` criou uma variável mutável que está atualmente vinculada a uma nova instância vazia de uma `String`. Ufa!

### Recebendo a entrada do usuário

Lembre-se de que incluímos a funcionalidade de entrada/saída da biblioteca padrão com `use std::io;` na primeira linha do programa. Agora chamaremos a função `stdin` do módulo `io`, que nos permitirá lidar com a entrada do usuário:

```rust
io::stdin()
  .read_line(&mut guess)
```

Se não tivéssemos importado o módulo `io` com `use std::io;` no início do programa, ainda poderíamos usar a função escrevendo esta chamada de função como `std::io::stdin`. A função `stdin` retorna uma instância de `std::io::Stdin`, que é um tipo que representa um identificador para a entrada padrão do terminal.

Em seguida, a linha `.read_line(&mut guess)` chama o método `read_line` no manipulador da entrada padrão para ler o que o usuário digitar no teclado. Também estamos passando `&mut guess` como argumento para `read_line`, indicando em qual *String* o texto digitado pelo usuário deve ser armazenado. O trabalho completo de `read_line` é pegar tudo o que o usuário digitar na entrada padrão e acrescentar isso a uma *String* (sem sobrescrever o conteúdo que ela já possui), por isso passamos essa *String* como argumento. O argumento precisa ser mutável para que o método possa alterar o conteúdo da *String*.

O símbolo `&` indica que esse argumento é uma referência, o que permite que várias partes do código acessem o mesmo dado sem precisar copiá-lo várias vezes na memória. Referências são um recurso mais avançado, e uma das grandes vantagens do Rust é justamente o quanto ele torna o uso de referências seguro e simples. Você não precisa entender todos esses detalhes para finalizar este programa. Por enquanto, o mais importante é saber que, assim como as variáveis, referências são imutáveis por padrão. Por isso, é necessário escrever `&mut guess` em vez de `&guess` para permitir que o valor seja modificado. (O Capítulo 4 explicará referências com mais detalhes.)

### Tratando falha potencial com Result

Ainda estamos analisando essa linha de código. A próxima parte que vamos ver é este método:

```rust
.expect("Failed to read line");
```

Poderíamos ter escrito este código como:

```rust
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

No entanto, uma linha muito longa fica difícil de ler, então é melhor dividi-la. Geralmente é uma boa ideia adicionar uma quebra de linha e outros espaços em branco para facilitar a leitura quando você chama um método usando a sintaxe `.method_name()`. Agora vamos falar sobre o que essa linha faz.

Como foi mencionado antes, `read_line` coloca tudo o que o usuário digitar dentro da string que passamos para ele, mas esse método também retorna um valor do tipo `Result`. `Result` é uma enumeração (geralmente chamada apenas de *enum*), que é um tipo que pode estar em um de vários estados possíveis. Cada um desses estados é chamado de *variant* (variante).

O Capítulo 6 explicará enums com mais detalhes. O objetivo do tipo `Result` é representar informações relacionadas ao tratamento de erros.

As variantes de `Result` são `Ok` e `Err`. A variante `Ok` indica que a operação foi bem-sucedida e contém o valor gerado com sucesso. Já a variante `Err` indica que a operação falhou e contém informações sobre como ou por que ela falhou.

Valores do tipo `Result`, assim como valores de qualquer outro tipo, possuem métodos definidos para eles. Uma instância de `Result` tem um método chamado `expect`, que você pode chamar.

Se essa instância de `Result` for um valor `Err`, o método `expect` fará o programa encerrar a execução e exibirá a mensagem que você passou como argumento para ele. Se o método `read_line` retornar um `Err`, isso provavelmente será resultado de algum erro vindo do sistema operacional.

Se essa instância de `Result` for um valor `Ok`, o método `expect` vai extrair o valor que está dentro do `Ok` e retorná-lo para que você possa usá-lo. Nesse caso, esse valor é o número de bytes da entrada digitada pelo usuário.

Se você não chamar `expect`, o programa ainda vai compilar, mas você receberá um aviso (*warning*).

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `Result` that must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: this `Result` may be an `Err` variant, which should be handled
   = note: `#[warn(unused_must_use)]` on by default
help: use `let _ = ...` to ignore the resulting value
   |
10 |     let _ = io::stdin().read_line(&mut guess);
   |     +++++++

warning: `guessing_game` (bin "guessing_game") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.59s
```

Rust avisa que você não usou o valor `Result` retornado de `read_line`, indicando que o programa não tratou um possível erro.

A maneira correta de suprimir o aviso é realmente escrever código de tratamento de erros, mas no nosso caso queremos apenas travar este programa quando um problema ocorrer, então podemos usar `expect`. Você aprenderá sobre recuperação de erros no Capítulo 9.

### Imprimindo valores com placeholders do `println!`

Além da chave de fechamento, há apenas mais uma linha para discutir no código até agora:

```rust
println!("You guessed: {guess}");
```

Essa linha imprime a string que agora contém a entrada do usuário. O par de chaves `{}` é um *placeholder*: pense nele como pequenas "pinças de caranguejo" que seguram um valor no lugar.

Ao imprimir o valor de uma variável, você pode colocar o nome da variável dentro das chaves. Ao imprimir o resultado de uma expressão, coloque chaves vazias na string e, em seguida, passe uma lista de expressões separadas por vírgula, correspondendo a cada placeholder na mesma ordem.

Por exemplo, imprimir uma variável e o resultado de uma expressão em uma única chamada de `println!` ficaria assim:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Esse código iria imprimir `x = 5 and y + 2 = 12`.

### Testando a primeira parte

Vamos testar a primeira parte do jogo de adivinhação. Execute-o usando o comando `cargo run`:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Neste ponto, a primeira parte do jogo está concluída: estamos recebendo a entrada do teclado e a imprimindo.

## Gerando um número secreto

Em seguida, precisamos gerar um número secreto que o usuário tentará adivinhar. O número secreto deve ser diferente toda vez para que o jogo seja divertido de jogar mais de uma vez. Usaremos um número aleatório entre 1 e 100 para que o jogo não seja muito difícil. Rust ainda não inclui funcionalidade de número aleatório em sua biblioteca padrão. No entanto, a equipe do Rust fornece um crate `rand` com essa funcionalidade.

### Aumentando a funcionalidade com um crate

Lembre-se de que um crate é uma coleção de arquivos de código-fonte Rust. O projeto que estivemos construindo é um crate binário, que é um executável. O crate `rand` é um crate de biblioteca, que contém código destinado a ser usado em outros programas e não pode ser executado por conta própria.

A coordenação de crates externos pelo Cargo é onde o Cargo realmente brilha. Antes de podermos escrever código que usa `rand`, precisamos modificar o arquivo *Cargo.toml* para incluir o crate `rand` como uma dependência. Abra esse arquivo agora e adicione a seguinte linha ao final, abaixo do cabeçalho da seção `[dependencies]` que o Cargo criou para você. Certifique-se de especificar `rand` exatamente como temos aqui, com este número de versão, ou os exemplos de código neste tutorial podem não funcionar:

Arquivo: Cargo.toml
```toml
[dependencies]
rand = "0.8.5"
```

No arquivo *Cargo.toml*, tudo que segue um cabeçalho é parte daquela seção que continua até que outra seção comece. Em `[dependencies]`, você diz ao Cargo quais crates externos seu projeto depende e quais versões desses crates você requer. Neste caso, especificamos o crate `rand` com o especificador de versão semântica `0.8.5`. O Cargo entende Versionamento Semântico (às vezes chamado de *SemVer*), que é um padrão para escrever números de versão. O especificador `0.8.5` é na verdade uma abreviação para `^0.8.5`, que significa qualquer versão que seja pelo menos `0.8.5`, mas abaixo de `0.9.0`.

O Cargo considera que essas versões possuem APIs públicas compatíveis com a versão `0.8.5`, e esta especificação garante que você obterá a versão de correção mais recente que ainda compilará com o código deste capítulo. Não há garantia de que qualquer versão `0.9.0` ou superior tenha a mesma API que a utilizada nos exemplos a seguir.

Agora, sem alterar nada do código, vamos compilar o projeto, como mostrado na Listagem 2-2.

```bash
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```
[Listagem 2-2](#listagem-2-2): A saída de executar `cargo build` após adicionar o crate `rand` como uma dependência

Você pode ver números de versão diferentes (mas todos serão compatíveis com o código, graças ao SemVer!) e linhas diferentes (dependendo do sistema operacional), e as linhas podem estar em uma ordem diferente.

Quando incluímos uma dependência externa, o Cargo busca as versões mais recentes de tudo que a dependência precisa do registro, que é uma cópia de dados do Crates.io. Crates.io é onde as pessoas no ecossistema Rust publicam seus projetos Rust de código aberto para outros usarem.

Após atualizar o registro, o Cargo verifica a seção `[dependencies]` e baixa quaisquer crates listados que ainda não foram baixados. Neste caso, embora tenhamos listado apenas `rand` como uma dependência, o Cargo também pegou outros crates dos quais `rand` depende para funcionar. Após baixar os crates, Rust os compila e então compila o projeto com as dependências disponíveis.

Se você executar `cargo build` imediatamente novamente sem fazer nenhuma alteração, não obterá nenhuma saída além da linha `Finished`. O Cargo sabe que já baixou e compilou as dependências, e você não mudou nada sobre elas no seu arquivo *Cargo.toml*. O Cargo também sabe que você não mudou nada sobre seu código, então não recompila isso também. Sem nada para fazer, ele simplesmente encerra.

Se você abrir o arquivo *src/main.rs*, fizer uma mudança trivial, e então salvá-lo e compilar novamente, você verá apenas duas linhas de saída:

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Essas linhas mostram que o Cargo atualiza apenas a compilação com sua pequena mudança no arquivo *src/main.rs*. Suas dependências não mudaram, então o Cargo sabe que pode reutilizar o que já baixou e compilou para elas.

#### Garantindo compilações reproduzíveis

O Cargo tem um mecanismo que garante que você consiga reconstruir exatamente o mesmo programa sempre que você — ou qualquer outra pessoa — compilar o seu código. Para isso, o Cargo usa apenas as versões das dependências que você definiu, até que você diga explicitamente que quer atualizar.

Por exemplo: imagine que, na próxima semana, seja lançada a versão `0.8.6` da crate `rand`. Essa versão corrige um bug importante, mas também introduz um problema novo que quebra o seu código. Para evitar esse tipo de surpresa, o Rust cria o arquivo `Cargo.lock` na primeira vez que você executa `cargo build`. É por isso que agora esse arquivo aparece no diretório `guessing_game`.

Quando você compila um projeto pela primeira vez, o Cargo descobre quais versões das dependências atendem aos requisitos do projeto e grava essas versões no arquivo `Cargo.lock`. Nas próximas compilações, o Cargo percebe que esse arquivo já existe e usa exatamente as versões registradas ali, em vez de recalcular tudo novamente.

Isso garante que a compilação seja reprodutível, ou seja, o resultado será sempre o mesmo. Em outras palavras, o seu projeto continuará usando a versão `0.8.5` até que você decida atualizar manualmente, graças ao `Cargo.lock`.

Como o arquivo `Cargo.lock` é essencial para garantir builds reproduzíveis, normalmente ele é incluído no controle de versão junto com o restante do código do projeto.

#### Atualizando um crate para obter uma nova versão

Quando você realmente quiser atualizar um crate, o Cargo fornece o comando `update`, que ignorará o arquivo *Cargo.lock* e descobrirá todas as versões mais recentes que se encaixam nas suas especificações no *Cargo.toml*. O Cargo então escreverá essas versões no arquivo *Cargo.lock*. Caso contrário, por padrão, o Cargo procurará apenas versões maiores que `0.8.5` e menores que `0.9.0`. Se o crate `rand` lançou as duas novas versões `0.8.6` e `0.999.0`, você veria o seguinte se executasse `cargo update`:

```bash
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

O Cargo ignora o lançamento `0.999.0`. Neste ponto, você também notaria uma mudança no seu arquivo *Cargo.lock* observando que a versão do crate `rand` que você está usando agora é `0.8.6`. Para usar a versão `0.999.0` do `rand` ou qualquer versão na série `0.999.x`, você teria que atualizar o arquivo *Cargo.toml* para ficar assim (não faça essa mudança na verdade porque os exemplos a seguir assumem que você está usando `rand 0.8`):

```toml
[dependencies]
rand = "0.999.0"
```

Da próxima vez que você executar `cargo build`, o Cargo atualizará o registro de crates disponíveis e reavaliará seus requisitos de `rand` de acordo com a nova versão que você especificou.

Há muito mais a dizer sobre o [Cargo](https://doc.rust-lang.org/cargo/) e [seu ecossistema](https://doc.rust-lang.org/cargo/reference/publishing.html), que discutiremos no Capítulo 14, mas por enquanto, isso é tudo que você precisa saber. O Cargo torna muito fácil reutilizar bibliotecas, então os Rustáceos conseguem escrever projetos menores que são montados a partir de vários pacotes.

### Gerando um número aleatório

Vamos começar a usar `rand` para gerar um número para adivinhar. O próximo passo é atualizar *src/main.rs*, como mostrado na Listagem 2-3.

Arquivo: src/main.rs
```rust
use std::io;

use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```
[Listagem 2-3](#listagem-2-3): Adicionando código para gerar um número aleatório

Primeiro, adicionamos a linha `use rand::Rng;`. A trait `Rng` define métodos que geradores de números aleatórios implementam, e essa trait deve estar no escopo para usarmos esses métodos. O Capítulo 10 cobrirá traits em detalhes.

Em seguida, estamos adicionando duas linhas no meio. Na primeira linha, chamamos a função `rand::thread_rng` que nos dá o gerador de números aleatórios particular que vamos usar: um que é local à thread atual de execução e é semeado pelo sistema operacional. Então, chamamos o método `gen_range` no gerador de números aleatórios. Este método é definido pela trait `Rng` que trouxemos para o escopo com a declaração `use rand::Rng;`. O método `gen_range` recebe uma expressão de intervalo como argumento e gera um número aleatório no intervalo. O tipo de expressão de intervalo que estamos usando aqui tem a forma `start..=end` e é inclusivo nos limites inferior e superior, então precisamos especificar `1..=100` para solicitar um número entre 1 e 100.

> **Nota:** Você não saberá simplesmente quais traits usar e quais métodos e funções chamar de um crate, então cada crate tem documentação com instruções para usá-lo. Outro recurso legal do Cargo é que executar o comando `cargo doc --open` construirá a documentação fornecida por todas as suas dependências localmente e a abrirá no seu navegador. Se você estiver interessado em outras funcionalidades do crate `rand`, por exemplo, execute `cargo doc --open` e clique em `rand` na barra lateral à esquerda.

A segunda nova linha imprime o número secreto. Isso é útil enquanto estamos desenvolvendo o programa para poder testá-lo, mas o deletaremos da versão final. Não é muito jogo se o programa imprime a resposta assim que começa!

Tente executar o programa algumas vezes:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Você deve obter números aleatórios diferentes, e todos devem ser números entre 1 e 100. Ótimo trabalho!

## Comparando o palpite com o número secreto

Agora que temos entrada do usuário e um número aleatório, podemos compará-los. Esse passo é mostrado na Listagem 2-4. Observe que este código não compilará ainda, como explicaremos.

Arquivo: src/main.rs (Este código não compila!)
```rust
use std::cmp::Ordering;
use std::io;
use rand::Rng;

fn main() {
  // --snip--
  println!("You guessed: {guess}");
  
  match guess.cmp(&secret_number) {
    Ordering::Less => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal => println!("You win!"),
  }
}
```
[Listagem 2-4](#listagem-2-4): Tratando os possíveis valores de retorno de comparar dois números

Primeiro, adicionamos outra declaração `use`, trazendo um tipo chamado `std::cmp::Ordering` para o escopo da biblioteca padrão. O tipo `Ordering` é outro enum e tem as variantes `Less`, `Greater` e `Equal`. Esses são os três resultados possíveis quando você compara dois valores.

Então, adicionamos cinco novas linhas no final que usam o tipo `Ordering`. O método `cmp` compara dois valores e pode ser chamado em qualquer coisa que possa ser comparada. Ele recebe uma referência para o que quer que você queira comparar: aqui, está comparando `guess` com `secret_number`. Então, retorna uma variante do enum `Ordering` que trouxemos para o escopo com a declaração `use`. Usamos uma expressão `match` para decidir o que fazer a seguir com base em qual variante de `Ordering` foi retornada da chamada para `cmp` com os valores em `guess` e `secret_number`.

Uma expressão `match` é composta de braços (*arms*). Um braço consiste de um padrão para comparar, e o código que deve ser executado se o valor dado ao `match` se encaixar no padrão daquele braço. Rust pega o valor dado ao `match` e procura através do padrão de cada braço por vez. Padrões e a construção `match` são recursos poderosos do Rust: eles permitem que você expresse uma variedade de situações que seu código pode encontrar, e garantem que você trate todas elas. Esses recursos serão cobertos em detalhes no Capítulo 6 e no Capítulo 19, respectivamente.

Vamos percorrer um exemplo com a expressão `match` que usamos aqui. Digamos que o usuário adivinhou 50 e o número secreto gerado aleatoriamente desta vez é 38.

Quando o código compara 50 a 38, o método `cmp` retornará `Ordering::Greater` porque 50 é maior que 38. A expressão `match` obtém o valor `Ordering::Greater` e começa a verificar o padrão de cada braço. Ela olha para o padrão do primeiro braço, `Ordering::Less`, e vê que o valor `Ordering::Greater` não corresponde a `Ordering::Less`, então ignora o código naquele braço e passa para o próximo braço. O padrão do próximo braço é `Ordering::Greater`, que corresponde! O código associado naquele braço será executado e imprimirá `Too big!` na tela. A expressão `match` termina após a primeira correspondência bem-sucedida, então ela não olhará para o último braço neste cenário.

No entanto, o código na Listagem 2-4 ainda não compilará. Vamos tentar:

```bash
$ cargo build
   Compiling libc v0.2.86
   Compiling getrandom v0.2.2
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.10
   Compiling rand_core v0.6.2
   Compiling rand_chacha v0.3.0
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:23:21
   |
23 |     match guess.cmp(&secret_number) {
   |                 --- ^^^^^^^^^^^^^^ expected `&String`, found `&{integer}`
   |                 |
   |                 arguments to this method are incorrect
   |
   = note: expected reference `&String`
              found reference `&{integer}`
note: method defined here
  --> /rustc/1159e78c4747b02ef996e55082b704c09b970588/library/core/src/cmp.rs:979:8

For more information about this error, try `rustc --explain E0308`.
error: could not compile `guessing_game` (bin "guessing_game") due to 1 previous error
```

O núcleo do erro afirma que há tipos incompatíveis. Rust tem um sistema de tipos forte e estático. No entanto, também tem inferência de tipos. Quando escrevemos `let mut guess = String::new()`, Rust foi capaz de inferir que `guess` deveria ser uma `String` e não nos fez escrever o tipo. O `secret_number`, por outro lado, é um tipo numérico. Alguns dos tipos numéricos do Rust podem ter um valor entre 1 e 100: `i32`, um número de 32 bits; `u32`, um número sem sinal de 32 bits; `i64`, um número de 64 bits; bem como outros. A menos que especificado de outra forma, Rust assume um `i32` por padrão, que é o tipo de `secret_number` a menos que você adicione informações de tipo em outro lugar que fariam Rust inferir um tipo numérico diferente. A razão do erro é que Rust não pode comparar uma string e um tipo numérico.

Em última análise, queremos converter a `String` que o programa lê como entrada em um tipo numérico para que possamos compará-la numericamente ao número secreto. Fazemos isso adicionando esta linha ao corpo da função `main`:

Arquivo: src/main.rs
```rust
    // --snip--

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
```

A linha é:

```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Criamos uma variável chamada `guess`. Mas espere, o programa já não tem uma variável chamada `guess`? Tem, mas o Rust nos permite de forma útil sombrear (*shadow*) o valor anterior de `guess` com um novo. Sombreamento nos permite reutilizar o nome da variável `guess` em vez de nos forçar a criar duas variáveis únicas, como `guess_str` e `guess`, por exemplo. Cobriremos isso em mais detalhes no Capítulo 3, mas por enquanto, saiba que esse recurso é frequentemente usado quando você quer converter um valor de um tipo para outro tipo.

Vinculamos esta nova variável à expressão `guess.trim().parse()`. O `guess` na expressão se refere à variável `guess` original que continha a entrada como uma string. O método `trim` em uma instância de `String` eliminará qualquer espaço em branco no início e no fim, o que devemos fazer antes de podermos converter a string em um `u32`, que só pode conter dados numéricos. O usuário deve pressionar enter para satisfazer `read_line` e inserir seu palpite, o que adiciona um caractere de nova linha à string. Por exemplo, se o usuário digita 5 e pressiona enter, `guess` fica assim: `5\n`. O `\n` representa "nova linha". (No Windows, pressionar enter resulta em um retorno de carro e uma nova linha, `\r\n`.) O método `trim` elimina `\n` ou `\r\n`, resultando em apenas `5`.

O método `parse` em strings converte uma string em outro tipo. Aqui, o usamos para converter de uma string em um número. Precisamos dizer ao Rust o tipo exato de número que queremos usando `let guess: u32`. Os dois pontos (`:`) após `guess` dizem ao Rust que anotaremos o tipo da variável. Rust tem alguns tipos numéricos embutidos; o `u32` visto aqui é um inteiro sem sinal de 32 bits. É uma boa escolha padrão para um número positivo pequeno. Você aprenderá sobre outros tipos numéricos no Capítulo 3.

Adicionalmente, a anotação `u32` neste programa de exemplo e a comparação com `secret_number` significa que Rust inferirá que `secret_number` também deve ser um `u32`. Então, agora a comparação será entre dois valores do mesmo tipo!

O método `parse` só funcionará em caracteres que podem logicamente ser convertidos em números e, portanto, pode facilmente causar erros. Se, por exemplo, a string contivesse `A👍%`, não haveria maneira de converter isso em um número. Como pode falhar, o método `parse` retorna um tipo `Result`, assim como o método `read_line` faz (discutido anteriormente em "Tratando falha potencial com Result"). Trataremos este `Result` da mesma forma usando o método `expect` novamente. Se `parse` retornar uma variante `Err` de `Result` porque não conseguiu criar um número a partir da string, a chamada `expect` travará o jogo e imprimirá a mensagem que damos a ela. Se `parse` puder converter com sucesso a string em um número, ele retornará a variante `Ok` de `Result`, e `expect` retornará o número que queremos do valor `Ok`.

Vamos executar o programa agora:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Legal! Mesmo que espaços tenham sido adicionados antes do palpite, o programa ainda descobriu que o usuário adivinhou 76. Execute o programa algumas vezes para verificar o comportamento diferente com diferentes tipos de entrada: adivinhe o número corretamente, adivinhe um número muito alto e adivinhe um número muito baixo.

Temos a maior parte do jogo funcionando agora, mas o usuário pode fazer apenas um palpite. Vamos mudar isso adicionando um loop!

## Permitindo múltiplos palpites com looping

A palavra-chave `loop` cria um loop infinito. Adicionaremos um loop para dar aos usuários mais chances de adivinhar o número:

Arquivo: src/main.rs
```rust
  // --snip--

  println!("The secret number is: {secret_number}");

  loop {
    println!("Please input your guess.");

    // --snip--

    match guess.cmp(&secret_number) {
      Ordering::Less => println!("Too small!"),
      Ordering::Greater => println!("Too big!"),
      Ordering::Equal => println!("You win!"),
    }
  }
}
```

Como você pode ver, movemos tudo do prompt de entrada de palpite em diante para dentro de um loop. Certifique-se de indentar as linhas dentro do loop mais quatro espaços cada e execute o programa novamente. O programa agora pedirá outro palpite para sempre, o que na verdade introduz um novo problema. Não parece que o usuário pode sair!

O usuário sempre poderia interromper o programa usando o atalho de teclado Ctrl+C. Mas há outra maneira de escapar deste monstro insaciável, como mencionado na discussão sobre `parse` em "Comparando o Palpite com o Número Secreto": se o usuário inserir uma resposta não numérica, o programa travará. Podemos aproveitar isso para permitir que o usuário saia, como mostrado aqui:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Digitar `quit` sairá do jogo, mas como você notará, também sairá ao inserir qualquer outra entrada não numérica. Isso é subótimo, no mínimo; queremos que o jogo também pare quando o número correto for adivinhado.

### Saindo após um palpite correto

Vamos programar o jogo para sair quando o usuário ganhar adicionando uma declaração `break`:

Arquivo: src/main.rs
```rust
        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

Adicionar a linha `break` após `You win!` faz o programa sair do loop quando o usuário adivinha o número secreto corretamente. Sair do loop também significa sair do programa, porque o loop é a última parte de `main`.

### Tratando entrada inválida

Para refinar ainda mais o comportamento do jogo, em vez de travar o programa quando o usuário insere um não número, vamos fazer o jogo ignorar um não número para que o usuário possa continuar adivinhando. Podemos fazer isso alterando a linha onde `guess` é convertido de uma `String` para um `u32`, como mostrado na Listagem 2-5.

Arquivo: src/main.rs
```rust
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // --snip--
```
[Listagem 2-5](#listagem-2-5): Ignorando um palpite não numérico e pedindo outro palpite em vez de travar o programa

Mudamos de uma chamada `expect` para uma expressão `match` para passar de travar em um erro para tratar o erro. Lembre-se de que `parse` retorna um tipo `Result` e `Result` é um enum que tem as variantes `Ok` e `Err`. Estamos usando uma expressão `match` aqui, como fizemos com o resultado `Ordering` do método `cmp`.

Se `parse` conseguir transformar com sucesso a string em um número, retornará um valor `Ok` que contém o número resultante. Esse valor `Ok` corresponderá ao padrão do primeiro braço, e a expressão `match` retornará apenas o valor `num` que `parse` produziu e colocou dentro do valor `Ok`. Esse número acabará exatamente onde queremos na nova variável `guess` que estamos criando.

Se `parse` não conseguir transformar a string em um número, retornará um valor `Err` que contém mais informações sobre o erro. O valor `Err` não corresponde ao padrão `Ok(num)` no primeiro braço de match, mas corresponde ao padrão `Err(_)` no segundo braço. O sublinhado, `_`, é um valor coringa; neste exemplo, estamos dizendo que queremos corresponder a todos os valores `Err`, não importa qual informação eles tenham dentro deles. Então, o programa executará o código do segundo braço, `continue`, que diz ao programa para ir para a próxima iteração do loop e pedir outro palpite. Efetivamente, o programa ignora todos os erros que `parse` possa encontrar!

Agora tudo no programa deve funcionar como esperado. Vamos tentar:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Incrível! Com um pequeno ajuste final, terminaremos o jogo de adivinhação. Lembre-se de que o programa ainda está imprimindo o número secreto. Isso funcionou bem para testes, mas estraga o jogo. Vamos deletar o `println!` que mostra o número secreto. A Listagem 2-6 mostra o código final.

Arquivo: src/main.rs
```rust
use std::cmp::Ordering;
use std::io;

use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```
[Listagem 2-6](#listagem-2-6): Código completo do jogo de adivinhação

Neste ponto, você construiu com sucesso o jogo de adivinhação. Parabéns!

## Resumo

Este projeto foi uma maneira prática de apresentá-lo a muitos conceitos novos do Rust: `let`, `match`, funções, o uso de crates externos e mais. Nos próximos capítulos, você aprenderá sobre esses conceitos em mais detalhes. O Capítulo 3 cobre conceitos que a maioria das linguagens de programação tem, como variáveis, tipos de dados e funções, e mostra como usá-los em Rust. O Capítulo 4 explora ownership, um recurso que torna Rust diferente de outras linguagens. O Capítulo 5 discute structs e sintaxe de método, e o Capítulo 6 explica como enums funcionam.