---
title: "Melhorando tratamento de erros e modularidade"
chapter_code: 12-03
slug: melhorando-tratamento-de-erros-e-modularidade
---

# Refatorando para Melhorar Modularidade e Tratamento de Erros

Para melhorar nosso programa, corrigiremos quatro problemas relacionados à estrutura do programa e à forma como ele lida com erros potenciais. Primeiro, nossa função `main` agora executa duas tarefas: faz parse dos argumentos e lê arquivos. À medida que nosso programa crescer, o número de tarefas separadas que a função `main` trata aumentará. À medida que uma função ganha responsabilidades, fica mais difícil raciocinar sobre ela, mais difícil testá-la e mais difícil mudá-la sem quebrar uma de suas partes. É melhor separar a funcionalidade para que cada função seja responsável por uma tarefa.

Este problema também se relaciona ao segundo: embora `query` e `file_path` sejam variáveis de configuração do nosso programa, variáveis como `contents` são usadas para executar a lógica do programa. Quanto mais longa `main` se tornar, mais variáveis precisaremos trazer para o escopo; quanto mais variáveis tivermos no escopo, mais difícil será acompanhar o propósito de cada uma. É melhor agrupar as variáveis de configuração em uma estrutura para deixar seu propósito claro.

O terceiro problema é que usamos `expect` para imprimir uma mensagem de erro quando a leitura do arquivo falha, mas a mensagem de erro apenas imprime `Should have been able to read the file`. Ler um arquivo pode falhar de várias formas: por exemplo, o arquivo pode estar ausente, ou podemos não ter permissão para abri-lo. No momento, independentemente da situação, imprimiríamos a mesma mensagem de erro para tudo, o que não daria ao usuário nenhuma informação!

O quarto problema é que usamos `expect` para lidar com um erro e, se o usuário executar nosso programa sem especificar argumentos suficientes, obterá um erro `index out of bounds` do Rust que não explica claramente o problema. Seria melhor se todo o código de tratamento de erros estivesse em um lugar, para que futuros mantenedores tivessem apenas um lugar para consultar o código se a lógica de tratamento de erros precisasse mudar. Ter todo o código de tratamento de erros em um lugar também garantirá que imprimamos mensagens que façam sentido para nossos usuários finais.

Vamos abordar esses quatro problemas refatorando nosso projeto.

### Separando responsabilidades em projetos binários

O problema organizacional de alocar responsabilidade por várias tarefas à função `main` é comum em muitos projetos binários. Por isso, muitos programadores Rust acham útil dividir as responsabilidades separadas de um programa binário quando a função `main` começa a ficar grande. Este processo tem os seguintes passos:

- Dividir seu programa em um arquivo _main.rs_ e um arquivo _lib.rs_ e mover a lógica do programa para _lib.rs_.
- Enquanto sua lógica de parse de linha de comando for pequena, ela pode permanecer na função `main`.
- Quando a lógica de parse de linha de comando começar a ficar complicada, extraí-la da função `main` para outras funções ou tipos.

As responsabilidades que permanecem na função `main` após este processo devem se limitar ao seguinte:

- Chamar a lógica de parse de linha de comando com os valores dos argumentos
- Configurar qualquer outra configuração
- Chamar uma função `run` em _lib.rs_
- Lidar com o erro se `run` retornar um erro

Este padrão trata de separar responsabilidades: _main.rs_ lida com executar o programa e _lib.rs_ lida com toda a lógica da tarefa em questão. Como você não pode testar a função `main` diretamente, esta estrutura permite testar toda a lógica do programa movendo-a para fora da função `main`. O código que permanece na função `main` será pequeno o suficiente para verificar sua correção pela leitura. Vamos refazer nosso programa seguindo este processo.

#### Extraindo o parser de argumentos

Extrairemos a funcionalidade para fazer parse dos argumentos em uma função que `main` chamará. A Listagem 12-5 mostra o novo início da função `main` que chama uma nova função `parse_config`, que definiremos em _src/main.rs_.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, file_path) = parse_config(&args);

    println!("Searching for {query}");
    println!("In file {file_path}");

    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];

    (query, file_path)
}
```

<a id="listagem-12-5"></a>

[Listagem 12-5](#listagem-12-5): Extraindo uma função `parse_config` de `main`

Ainda estamos coletando os argumentos de linha de comando em um vetor, mas em vez de atribuir o valor do argumento no índice 1 à variável `query` e o valor no índice 2 à variável `file_path` dentro da função `main`, passamos o vetor inteiro para a função `parse_config`. A função `parse_config` então contém a lógica que determina qual argumento vai em qual variável e devolve os valores a `main`. Ainda criamos as variáveis `query` e `file_path` em `main`, mas `main` não tem mais a responsabilidade de determinar como os argumentos de linha de comando e as variáveis correspondem.

Esta refatoração pode parecer exagero para nosso programa pequeno, mas estamos refatorando em passos pequenos e incrementais. Depois desta mudança, execute o programa novamente para verificar que o parse de argumentos ainda funciona. É bom verificar seu progresso com frequência, para ajudar a identificar a causa de problemas quando eles ocorrem.

#### Agrupando valores de configuração

Podemos dar outro passo pequeno para melhorar ainda mais a função `parse_config`. No momento, estamos retornando uma tupla, mas logo em seguida desmontamos essa tupla em partes individuais novamente. Isso é um sinal de que talvez ainda não tenhamos a abstração certa.

Outro indicador de que há espaço para melhoria é a parte `config` de `parse_config`, que implica que os dois valores que retornamos estão relacionados e são ambos parte de um valor de configuração. Não estamos transmitindo este significado na estrutura dos dados além de agrupar os dois valores em uma tupla; em vez disso, colocaremos os dois valores em uma struct e daremos a cada campo da struct um nome significativo. Fazer isso facilitará para futuros mantenedores deste código entender como os diferentes valores se relacionam e qual é seu propósito.

A Listagem 12-6 mostra as melhorias na função `parse_config`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();

    Config { query, file_path }
}
```

<a id="listagem-12-6"></a>

[Listagem 12-6](#listagem-12-6): Refatorando `parse_config` para retornar uma instância de struct `Config`

Adicionamos uma struct chamada `Config` definida para ter campos chamados `query` e `file_path`. A assinatura de `parse_config` agora indica que retorna um valor `Config`. No corpo de `parse_config`, onde antes retornávamos slices de string que referenciam valores `String` em `args`, agora definimos `Config` para conter valores `String` com ownership. A variável `args` em `main` é a dona dos valores dos argumentos e só está emprestando à função `parse_config`; isso significa que violaríamos as regras de borrowing do Rust se `Config` tentasse tomar ownership dos valores em `args`.

Há várias formas de gerenciar os dados `String`; a rota mais fácil, embora um tanto ineficiente, é chamar o método `clone` nos valores. Isso fará uma cópia completa dos dados para a instância de `Config` possuir, o que leva mais tempo e memória do que armazenar uma referência aos dados da string. Porém, clonar os dados também torna nosso código muito direto, porque não precisamos gerenciar lifetimes das referências; nesta circunstância, abrir mão de um pouco de desempenho para ganhar simplicidade é uma troca que vale a pena.

> ### As trocas de usar `clone`
>
> Há uma tendência entre muitos Rustaceans de evitar usar `clone` para corrigir problemas de ownership por causa de seu custo em tempo de execução. No Capítulo 13, você aprenderá como usar métodos mais eficientes neste tipo de situação. Mas, por enquanto, está ok copiar algumas strings para continuar progredindo, porque você fará essas cópias apenas uma vez e sua string de consulta e caminho de arquivo são bem pequenos. É melhor ter um programa funcionando que seja um pouco ineficiente do que tentar hiperotimizar o código na primeira passagem. À medida que ganhar mais experiência com Rust, será mais fácil começar com a solução mais eficiente; por enquanto, é perfeitamente aceitável chamar `clone`.

Atualizamos `main` para colocar a instância de `Config` retornada por `parse_config` em uma variável chamada `config`, e atualizamos o código que antes usava as variáveis separadas `query` e `file_path` para usar os campos na struct `Config`.

Agora nosso código transmite mais claramente que `query` e `file_path` estão relacionados e que seu propósito é configurar como o programa funcionará. Qualquer código que use esses valores sabe que deve encontrá-los na instância `config` nos campos nomeados para seu propósito.

#### Criando um construtor para `Config`

Até agora, extraímos a lógica responsável por fazer parse dos argumentos de linha de comando de `main` e a colocamos na função `parse_config`. Isso nos ajudou a ver que os valores `query` e `file_path` estavam relacionados e que essa relação deveria ser transmitida em nosso código. Em seguida, adicionamos uma struct `Config` para nomear o propósito relacionado de `query` e `file_path` e para poder retornar os nomes dos valores como nomes de campos da struct a partir da função `parse_config`.

Então, agora que o propósito da função `parse_config` é criar uma instância de `Config`, podemos mudar `parse_config` de uma função simples para uma função chamada `new` associada à struct `Config`. Fazer esta mudança tornará o código mais idiomático. Podemos criar instâncias de tipos na biblioteca padrão, como `String`, chamando `String::new`. Da mesma forma, ao mudar `parse_config` em uma função `new` associada a `Config`, poderemos criar instâncias de `Config` chamando `Config::new`. A Listagem 12-7 mostra as mudanças que precisamos fazer.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

<a id="listagem-12-7"></a>

[Listagem 12-7](#listagem-12-7): Mudando `parse_config` para `Config::new`

Atualizamos `main` onde chamávamos `parse_config` para chamar `Config::new` em vez disso. Mudamos o nome de `parse_config` para `new` e o movemos para dentro de um bloco `impl`, que associa a função `new` a `Config`. Tente compilar este código novamente para garantir que funciona.

### Corrigindo o tratamento de erros

Agora trabalharemos em corrigir nosso tratamento de erros. Lembre-se de que tentar acessar os valores no vetor `args` nos índices 1 ou 2 fará o programa entrar em pânico se o vetor contiver menos de três itens. Tente executar o programa sem argumentos; ficará assim:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`

thread 'main' panicked at src/main.rs:27:21:
index out of bounds: the len is 1 but the index is 1
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

A linha `index out of bounds: the len is 1 but the index is 1` é uma mensagem de erro destinada a programadores. Não ajudará nossos usuários finais a entender o que deveriam fazer em vez disso. Vamos corrigir isso agora.

#### Melhorando a mensagem de erro

Na Listagem 12-8, adicionamos uma verificação na função `new` que verificará se o slice é longo o suficiente antes de acessar os índices 1 e 2. Se o slice não for longo o suficiente, o programa entra em pânico e exibe uma mensagem de erro melhor.

**Arquivo: src/main.rs**

```rust
impl Config {
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

<a id="listagem-12-8"></a>

[Listagem 12-8](#listagem-12-8): Adicionando uma verificação do número de argumentos

Este código é semelhante à função `Guess::new` que escrevemos na Listagem 9-13, onde chamamos `panic!` quando o argumento `value` estava fora do intervalo de valores válidos. Em vez de verificar um intervalo de valores aqui, verificamos se o comprimento de `args` é pelo menos `3`, e o restante da função pode operar assumindo que esta condição foi atendida. Se `args` tiver menos de três itens, esta condição será `true` e chamaremos a macro `panic!` para encerrar o programa imediatamente.

Com essas poucas linhas extras de código em `new`, vamos executar o programa sem argumentos novamente para ver como o erro parece agora:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`

thread 'main' panicked at src/main.rs:26:13:
not enough arguments
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Esta saída é melhor: agora temos uma mensagem de erro razoável. Porém, também temos informação extra que não queremos dar aos nossos usuários. Talvez a técnica que usamos na Listagem 9-13 não seja a melhor aqui: uma chamada a `panic!` é mais apropriada para um problema de programação do que de uso, como discutido no Capítulo 9. Em vez disso, usaremos a outra técnica que você aprendeu no Capítulo 9 — retornar um `Result` que indica sucesso ou erro.

#### Retornando um `Result` em vez de chamar `panic!`

Podemos em vez disso retornar um valor `Result` que conterá uma instância de `Config` no caso de sucesso e descreverá o problema no caso de erro. Também mudaremos o nome da função de `new` para `build`, porque muitos programadores esperam que funções `new` nunca falhem. Quando `Config::build` se comunicar com `main`, podemos usar o tipo `Result` para sinalizar que houve um problema. Então, podemos mudar `main` para converter uma variante `Err` em um erro mais prático para nossos usuários, sem o texto circundante sobre `thread 'main'` e `RUST_BACKTRACE` que uma chamada a `panic!` causa.

A Listagem 12-9 mostra as mudanças que precisamos fazer no valor de retorno da função que agora chamamos de `Config::build` e no corpo da função necessário para retornar um `Result`. Observe que isso não compilará até atualizarmos `main` também, o que faremos na próxima listagem.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-9"></a>

[Listagem 12-9](#listagem-12-9): Retornando um `Result` de `Config::build`

Nossa função `build` retorna um `Result` com uma instância de `Config` no caso de sucesso e um literal de string no caso de erro. Nossos valores de erro serão sempre literais de string que têm o lifetime `'static`.

Fizemos duas mudanças no corpo da função: em vez de chamar `panic!` quando o usuário não passa argumentos suficientes, agora retornamos um valor `Err`, e envolvemos o valor de retorno `Config` em um `Ok`. Essas mudanças fazem a função obedecer à sua nova assinatura de tipo.

Retornar um valor `Err` de `Config::build` permite que a função `main` lide com o valor `Result` retornado pela função `build` e saia do processo de forma mais limpa no caso de erro.

#### Chamando `Config::build` e lidando com erros

Para lidar com o caso de erro e imprimir uma mensagem amigável ao usuário, precisamos atualizar `main` para lidar com o `Result` retornado por `Config::build`, como mostrado na Listagem 12-10. Também tiraremos a responsabilidade de sair da ferramenta de linha de comando com código de erro diferente de zero de `panic!` e em vez disso implementaremos isso manualmente. Um status de saída diferente de zero é uma convenção para sinalizar ao processo que chamou nosso programa que o programa saiu em estado de erro.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-10"></a>

[Listagem 12-10](#listagem-12-10): Saindo com código de erro se construir um `Config` falhar

Nesta listagem, usamos um método que ainda não cobrimos em detalhes: `unwrap_or_else`, que é definido em `Result<T, E>` pela biblioteca padrão. Usar `unwrap_or_else` nos permite definir algum tratamento de erro personalizado, que não seja `panic!`. Se o `Result` for um valor `Ok`, o comportamento deste método é semelhante ao de `unwrap`: retorna o valor interno que `Ok` envolve. Porém, se o valor for um `Err`, este método chama o código no closure, que é uma função anônima que definimos e passamos como argumento a `unwrap_or_else`. Cobriremos closures com mais detalhes no Capítulo 13. Por enquanto, você só precisa saber que `unwrap_or_else` passará o valor interno do `Err`, que neste caso é o literal de string estático `"not enough arguments"` que adicionamos na Listagem 12-9, ao nosso closure no argumento `err` que aparece entre as barras verticais. O código no closure pode então usar o valor `err` quando executar.

Adicionamos uma nova linha `use` para trazer `process` da biblioteca padrão para o escopo. O código no closure que será executado no caso de erro tem apenas duas linhas: imprimimos o valor `err` e então chamamos `process::exit`. A função `process::exit` interromperá o programa imediatamente e retornará o número passado como código de status de saída. Isso é semelhante ao tratamento baseado em `panic!` que usamos na Listagem 12-8, mas não obtemos toda a saída extra. Vamos tentar:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

Ótimo! Esta saída é muito mais amigável para nossos usuários.

### Extraindo lógica de `main`

Agora que terminamos de refatorar o parse de configuração, vamos voltar à lógica do programa. Como afirmamos em Separando responsabilidades em projetos binários, extrairemos uma função chamada `run` que conterá toda a lógica atualmente na função `main` que não está envolvida em configurar configuração ou lidar com erros. Quando terminarmos, a função `main` será concisa e fácil de verificar por inspeção, e poderemos escrever testes para todo o restante da lógica.

A Listagem 12-11 mostra a pequena melhoria incremental de extrair uma função `run`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-11"></a>

[Listagem 12-11](#listagem-12-11): Extraindo uma função `run` contendo o restante da lógica do programa

A função `run` agora contém toda a lógica restante de `main`, começando pela leitura do arquivo. A função `run` recebe a instância de `Config` como argumento.

#### Retornando erros de `run`

Com a lógica restante do programa separada na função `run`, podemos melhorar o tratamento de erros, como fizemos com `Config::build` na Listagem 12-9. Em vez de permitir que o programa entre em pânico chamando `expect`, a função `run` retornará um `Result<T, E>` quando algo der errado. Isso nos permitirá consolidar ainda mais a lógica em torno do tratamento de erros em `main` de forma amigável ao usuário. A Listagem 12-12 mostra as mudanças que precisamos fazer na assinatura e no corpo de `run`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::error::Error;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    run(config);
}

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{contents}");

    Ok(())
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-12"></a>

[Listagem 12-12](#listagem-12-12): Mudando a função `run` para retornar `Result`

Fizemos três mudanças significativas aqui. Primeiro, mudamos o tipo de retorno da função `run` para `Result<(), Box<dyn Error>>`. Esta função antes retornava o tipo unitário `()`, e mantemos isso como o valor retornado no caso `Ok`.

Para o tipo de erro, usamos o trait object `Box<dyn Error>` (e trouxemos `std::error::Error` para o escopo com uma instrução `use` no topo). Cobriremos trait objects no Capítulo 18. Por enquanto, saiba apenas que `Box<dyn Error>` significa que a função retornará um tipo que implementa a trait `Error`, mas não precisamos especificar qual tipo concreto será o valor de retorno. Isso nos dá flexibilidade para retornar valores de erro que podem ser de tipos diferentes em casos de erro diferentes. A palavra-chave `dyn` é abreviação de _dynamic_.

Segundo, removemos a chamada a `expect` em favor do operador `?`, como falamos no Capítulo 9. Em vez de `panic!` em um erro, `?` retornará o valor de erro da função atual para o chamador lidar.

Terceiro, a função `run` agora retorna um valor `Ok` no caso de sucesso. Declaramos o tipo de sucesso da função `run` como `()` na assinatura, o que significa que precisamos envolver o valor do tipo unitário no valor `Ok`. Esta sintaxe `Ok(())` pode parecer um pouco estranha no início. Mas usar `()` assim é a forma idiomática de indicar que estamos chamando `run` apenas por seus efeitos colaterais; ela não retorna um valor que precisamos.

Quando você executa este código, ele compilará, mas exibirá um aviso:

```bash
$ cargo run -- the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
warning: unused `Result` that must be used
  --> src/main.rs:19:5
   |
19 |     run(config);
   |     ^^^^^^^^^^^
   |
   = note: this `Result` may be an `Err` variant, which should be handled
   = note: `#[warn(unused_must_use)]` on by default
help: use `let _ = ...` to ignore the resulting value
   |
19 |     let _ = run(config);
   |     +++++++

warning: `minigrep` (bin "minigrep") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.71s
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
...
```

O Rust nos diz que nosso código ignorou o valor `Result` e que o valor `Result` pode indicar que ocorreu um erro. Mas não estamos verificando se houve erro ou não, e o compilador nos lembra que provavelmente queríamos ter algum código de tratamento de erros aqui! Vamos corrigir esse problema agora.

#### Lidando com erros retornados de `run` em `main`

Verificaremos erros e os trataremos usando uma técnica semelhante à que usamos com `Config::build` na Listagem 12-10, mas com uma pequena diferença:

**Arquivo: src/main.rs**

```rust
    if let Err(e) = run(config) {
        println!("Application error: {e}");
        process::exit(1);
    }
```

Usamos `if let` em vez de `unwrap_or_else` para verificar se `run` retorna um valor `Err` e chamar `process::exit(1)` se retornar. A função `run` não retorna um valor que queremos `unwrap` da mesma forma que `Config::build` retorna a instância `Config`. Como `run` retorna `()` no caso de sucesso, só nos importamos em detectar um erro, então não precisamos de `unwrap_or_else` para retornar o valor desembrulhado, que seria apenas `()`.

Os corpos de `if let` e das funções `unwrap_or_else` são os mesmos nos dois casos: imprimimos o erro e saímos.

### Dividindo o código em uma crate de biblioteca

Nosso projeto `minigrep` está indo bem! Agora dividiremos o arquivo _src/main.rs_ e colocaremos algum código no arquivo _src/lib.rs_. Assim, podemos testar o código e ter um arquivo _src/main.rs_ com menos responsabilidades.

Vamos definir o código responsável por buscar texto em _src/lib.rs_ em vez de em _src/main.rs_, o que permitirá que nós (ou qualquer outra pessoa usando nossa biblioteca `minigrep`) chamemos a função de busca de mais contextos além do nosso binário `minigrep`.

Primeiro, vamos definir a assinatura da função `search` em _src/lib.rs_ como mostrado na Listagem 12-13, com um corpo que chama a macro `unimplemented!`. Explicaremos a assinatura com mais detalhes quando preenchermos a implementação.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    unimplemented!();
}
```

<a id="listagem-12-13"></a>

[Listagem 12-13](#listagem-12-13): Definindo a função `search` em _src/lib.rs_

Usamos a palavra-chave `pub` na definição da função para designar `search` como parte da API pública da nossa crate de biblioteca. Agora temos uma crate de biblioteca que podemos usar a partir da nossa crate binária e que podemos testar!

Agora precisamos trazer o código definido em _src/lib.rs_ para o escopo da crate binária em _src/main.rs_ e chamá-lo, como mostrado na Listagem 12-14.

**Arquivo: src/main.rs**

```rust
use minigrep::search;

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    for line in search(&config.query, &contents) {
        println!("{line}");
    }

    Ok(())
}
```

<a id="listagem-12-14"></a>

[Listagem 12-14](#listagem-12-14): Usando a função `search` da crate de biblioteca `minigrep` em _src/main.rs_

Adicionamos uma linha `use minigrep::search` para trazer a função `search` da crate de biblioteca para o escopo da crate binária. Depois, na função `run`, em vez de imprimir o conteúdo do arquivo, chamamos a função `search` e passamos `config.query` e `contents` como argumentos. Então, `run` usará um loop `for` para imprimir cada linha retornada por `search` que correspondeu à consulta. Este também é um bom momento para remover as chamadas `println!` na função `main` que exibiam a consulta e o caminho do arquivo, para que nosso programa imprima apenas os resultados da busca (se nenhum erro ocorrer).

Observe que a função de busca coletará todos os resultados em um vetor que retorna antes de qualquer impressão acontecer. Esta implementação pode ser lenta para exibir resultados ao buscar arquivos grandes, porque os resultados não são impressos conforme são encontrados; discutiremos uma forma possível de corrigir isso usando iterators no Capítulo 13.

Ufa! Foi muito trabalho, mas nos preparamos para o sucesso no futuro. Agora é muito mais fácil lidar com erros, e tornamos o código mais modular. Quase todo o nosso trabalho será feito em _src/lib.rs_ daqui em diante.

Vamos aproveitar esta nova modularidade fazendo algo que teria sido difícil com o código antigo, mas é fácil com o novo: escreveremos alguns testes!
