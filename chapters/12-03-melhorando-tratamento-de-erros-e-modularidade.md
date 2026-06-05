---
title: "Melhorando tratamento de erros e modularidade"
chapter_code: 12-03
slug: melhorando-tratamento-de-erros-e-modularidade
challenge_day: 15
reading_minutes: 47
---

# Refatorando para melhorar modularidade e tratamento de erros

Para melhorar nosso programa, vamos corrigir quatro problemas relacionados à estrutura do programa e à forma como ele lida com erros potenciais. Primeiro, nossa função `main` agora executa duas tarefas: analisa os argumentos e lê arquivos. À medida que nosso programa crescer, o número de tarefas separadas tratadas pela função `main` aumentará. Conforme uma função ganha responsabilidades, fica mais difícil raciocinar sobre ela, testá-la e alterá-la sem quebrar uma de suas partes. O melhor é separar a funcionalidade para que cada função seja responsável por uma tarefa.

Esse problema também se conecta ao segundo: embora `query` e `file_path` sejam variáveis de configuração do nosso programa, variáveis como `contents` são usadas para executar a lógica do programa. Quanto mais longa `main` se tornar, mais variáveis precisaremos trazer para o escopo; quanto mais variáveis tivermos no escopo, mais difícil será acompanhar o propósito de cada uma. É melhor agrupar as variáveis de configuração em uma única estrutura para deixar claro qual é o propósito delas.

O terceiro problema é que usamos `expect` para imprimir uma mensagem de erro quando a leitura do arquivo falha, mas a mensagem de erro apenas imprime `Erro ao ler o arquivo`. Ler um arquivo pode falhar de várias formas: por exemplo, o arquivo pode estar ausente, ou talvez não tenhamos permissão para abri-lo. Neste momento, independentemente da situação, imprimiríamos a mesma mensagem de erro para tudo, o que não daria nenhuma informação útil ao usuário!

O quarto problema é que usamos `expect` para lidar com um erro e, se o usuário executar nosso programa sem especificar argumentos suficientes, receberá um erro `index out of bounds` do Rust que não explica claramente o problema. Seria melhor se todo o código de tratamento de erros estivesse em um só lugar, para que futuros mantenedores tivessem apenas um ponto do código para consultar se a lógica de tratamento de erros precisasse mudar. Ter todo o código de tratamento de erros em um só lugar também ajudará a garantir que imprimamos mensagens significativas para nossos usuários finais.

Vamos abordar esses quatro problemas refatorando nosso projeto.

<a id="separando-responsabilidades-em-projetos-binarios"></a>

### Separando responsabilidades em projetos binários

O problema organizacional de atribuir responsabilidade por várias tarefas à função `main` é comum em muitos projetos binários. Por isso, muitos programadores Rust consideram útil separar as responsabilidades de um programa binário quando a função `main` começa a ficar grande. Esse processo tem os seguintes passos:

- Dividir seu programa em um arquivo _main.rs_ e um arquivo _lib.rs_ e mover a lógica do programa para _lib.rs_.
- Enquanto sua lógica de análise da linha de comando for pequena, ela pode permanecer na função `main`.
- Quando a lógica de análise da linha de comando começar a ficar complicada, extraí-la da função `main` para outras funções ou tipos.

As responsabilidades que permanecem na função `main` após esse processo devem se limitar ao seguinte:

- Chamar a lógica de análise da linha de comando com os valores dos argumentos
- Configurar qualquer outra opção necessária
- Chamar uma função `run` em _lib.rs_
- Lidar com o erro se `run` retornar um erro

Esse padrão trata de separar responsabilidades: _main.rs_ cuida da execução do programa, e _lib.rs_ cuida de toda a lógica da tarefa em questão. Como você não pode testar a função `main` diretamente, essa estrutura permite testar toda a lógica do programa ao movê-la para fora da função `main`. O código que permanecer na função `main` será pequeno o bastante para que possamos verificar sua correção por leitura. Vamos reestruturar nosso programa seguindo esse processo.

#### Extraindo o analisador de argumentos

Vamos extrair a funcionalidade de análise dos argumentos para uma função que `main` chamará. A Listagem 12-5 mostra o novo início da função `main`, que chama uma nova função `parse_config`, que definiremos em _src/main.rs_.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, file_path) = parse_config(&args);

    println!("Buscando {query}");
    println!("No arquivo {file_path}");

    let contents = fs::read_to_string(file_path)
        .expect("Erro ao ler o arquivo");

    println!("Com o texto:\n{contents}");
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];

    (query, file_path)
}
```

<a id="listagem-12-5"></a>

[Listagem 12-5](#listagem-12-5): Extraindo uma função `parse_config` de `main`

Ainda estamos coletando os argumentos de linha de comando em um vetor, mas, em vez de atribuir o valor do argumento no índice 1 à variável `query` e o valor do argumento no índice 2 à variável `file_path` dentro da função `main`, passamos o vetor inteiro para a função `parse_config`. A função `parse_config` então contém a lógica que determina qual argumento vai em qual variável e passa os valores de volta para `main`. Ainda criamos as variáveis `query` e `file_path` em `main`, mas `main` não tem mais a responsabilidade de determinar como os argumentos de linha de comando e as variáveis correspondem entre si.

Essa reestruturação pode parecer exagerada para nosso pequeno programa, mas estamos refatorando em passos pequenos e incrementais. Depois dessa mudança, execute o programa novamente para verificar se a análise dos argumentos ainda funciona. É bom verificar seu progresso com frequência, para ajudar a identificar a causa dos problemas quando eles aparecerem.

#### Agrupando valores de configuração

Podemos dar mais um pequeno passo para melhorar ainda mais a função `parse_config`. No momento, estamos retornando uma tupla, mas logo em seguida desmontamos essa tupla novamente em partes individuais. Isso é um sinal de que talvez ainda não tenhamos a abstração certa.

Outro indício de que há espaço para melhoria é a parte `config` de `parse_config`, que sugere que os dois valores retornados estão relacionados e fazem parte de um único valor de configuração. No momento, não estamos transmitindo esse significado na estrutura dos dados, além de agrupar os dois valores em uma tupla; em vez disso, colocaremos os dois valores em uma struct e daremos a cada campo da struct um nome significativo. Fazer isso facilitará para futuros mantenedores deste código entender como os diferentes valores se relacionam e qual é o propósito deles.

A Listagem 12-6 mostra as melhorias na função `parse_config`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Buscando {}", config.query);
    println!("No arquivo {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Erro ao ler o arquivo");

    println!("Com o texto:\n{contents}");
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

Adicionamos uma struct chamada `Config`, definida com campos chamados `query` e `file_path`. A assinatura de `parse_config` agora indica que ela retorna um valor `Config`. No corpo de `parse_config`, onde antes retornávamos slices de string que referenciavam valores `String` em `args`, agora definimos `Config` para conter valores `String` próprios. A variável `args` em `main` é a dona dos valores dos argumentos e apenas os empresta para a função `parse_config`; isso significa que violaríamos as regras de empréstimo do Rust se `Config` tentasse tomar posse dos valores em `args`.

Há várias formas de gerenciar os dados `String`; o caminho mais fácil, embora um tanto ineficiente, é chamar o método `clone` nos valores. Isso fará uma cópia completa dos dados para que a instância de `Config` tenha sua própria posse deles, o que leva mais tempo e usa mais memória do que armazenar uma referência aos dados da string. No entanto, clonar os dados também torna nosso código muito direto, porque não precisamos gerenciar os lifetimes das referências; nesta circunstância, abrir mão de um pouco de desempenho em troca de simplicidade é uma troca que vale a pena.

> ### As trocas de usar `clone`
>
> Há uma tendência entre muitos rustáceos de evitar usar `clone` para resolver problemas de ownership por causa de seu custo em tempo de execução. No [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures), você aprenderá a usar métodos mais eficientes nesse tipo de situação. Mas, por enquanto, não há problema em copiar algumas strings para continuar progredindo, porque você fará essas cópias apenas uma vez, e sua string de consulta e seu caminho de arquivo são bem pequenos. É melhor ter um programa funcionando, ainda que um pouco ineficiente, do que tentar hiperotimizar o código na primeira passada. À medida que ganhar mais experiência com Rust, será mais fácil começar pela solução mais eficiente; por enquanto, é perfeitamente aceitável chamar `clone`.

Atualizamos `main` para colocar a instância de `Config` retornada por `parse_config` em uma variável chamada `config`, e atualizamos o código que antes usava as variáveis separadas `query` e `file_path` para que agora use os campos da struct `Config`.

Agora nosso código comunica com mais clareza que `query` e `file_path` estão relacionados e que seu propósito é configurar como o programa funcionará. Qualquer código que use esses valores sabe que deve encontrá-los na instância `config`, em campos nomeados de acordo com seu propósito.

#### Criando um construtor para `Config`

Até agora, extraímos de `main` a lógica responsável por analisar os argumentos de linha de comando e a colocamos na função `parse_config`. Isso nos ajudou a ver que os valores `query` e `file_path` estavam relacionados, e que essa relação deveria ser comunicada em nosso código. Em seguida, adicionamos uma struct `Config` para nomear o propósito relacionado de `query` e `file_path` e para poder retornar os nomes dos valores como nomes de campos da struct a partir da função `parse_config`.

Agora que o propósito da função `parse_config` é criar uma instância de `Config`, podemos transformar `parse_config` de uma função comum em uma função chamada `new`, associada à struct `Config`. Fazer essa mudança tornará o código mais idiomático. Podemos criar instâncias de tipos da biblioteca padrão, como `String`, chamando `String::new`. De modo semelhante, ao transformar `parse_config` em uma função `new` associada a `Config`, poderemos criar instâncias de `Config` chamando `Config::new`. A Listagem 12-7 mostra as mudanças que precisamos fazer.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    println!("Buscando {}", config.query);
    println!("No arquivo {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Erro ao ler o arquivo");

    println!("Com o texto:\n{contents}");
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

Atualizamos `main` no ponto em que chamávamos `parse_config` para chamar `Config::new` em vez disso. Mudamos o nome de `parse_config` para `new` e a movemos para dentro de um bloco `impl`, que associa a função `new` a `Config`. Tente compilar este código novamente para garantir que ele funciona.

### Corrigindo o tratamento de erros

Agora vamos trabalhar na correção do nosso tratamento de erros. Lembre-se de que tentar acessar os valores no vetor `args` nos índices 1 ou 2 fará o programa entrar em pânico se o vetor contiver menos de três itens. Tente executar o programa sem argumentos; a saída será assim:

```console
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`

thread 'main' panicked at src/main.rs:27:21:
index out of bounds: the len is 1 but the index is 1
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

A linha `index out of bounds: the len is 1 but the index is 1` é uma mensagem de erro destinada a programadores. Ela não ajudará nossos usuários finais a entender o que deveriam fazer. Vamos corrigir isso agora.

#### Melhorando a mensagem de erro

Na Listagem 12-8, adicionamos uma verificação na função `new` que confirma se o slice é longo o suficiente antes de acessar os índices 1 e 2. Se o slice não for longo o suficiente, o programa entra em pânico e exibe uma mensagem de erro melhor.

**Arquivo: src/main.rs**

```rust
impl Config {
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!("argumentos insuficientes");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

<a id="listagem-12-8"></a>

[Listagem 12-8](#listagem-12-8): Adicionando uma verificação do número de argumentos

Este código é semelhante à função `Guess::new` que escrevemos na Listagem 9-13, onde chamamos `panic!` quando o argumento `value` estava fora do intervalo de valores válidos. Em vez de verificar um intervalo de valores aqui, verificamos se o comprimento de `args` é pelo menos `3`, e o restante da função pode operar sob a suposição de que essa condição foi atendida. Se `args` tiver menos de três itens, essa condição será `true`, e chamaremos a macro `panic!` para encerrar o programa imediatamente.

Com essas poucas linhas extras de código em `new`, vamos executar o programa sem argumentos novamente para ver como o erro parece agora:

```console
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`

thread 'main' panicked at src/main.rs:26:13:
argumentos insuficientes
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Essa saída é melhor: agora temos uma mensagem de erro razoável. Porém, também temos informações extras que não queremos mostrar aos nossos usuários. Talvez a técnica que usamos na Listagem 9-13 não seja a melhor aqui: uma chamada a `panic!` é mais apropriada para um problema de programação do que para um problema de uso, como discutido no [Capítulo 9](/livro/cap09-00-tratamento-de-erros). Em vez disso, usaremos a outra técnica que você aprendeu no [Capítulo 9](/livro/cap09-00-tratamento-de-erros): retornar um `Result` que indique sucesso ou erro.

#### Retornando um `Result` em vez de chamar `panic!`

Em vez disso, podemos retornar um valor `Result` que conterá uma instância de `Config` no caso de sucesso e descreverá o problema no caso de erro. Também vamos mudar o nome da função de `new` para `build`, porque muitos programadores esperam que funções `new` nunca falhem. Quando `Config::build` se comunicar com `main`, podemos usar o tipo `Result` para sinalizar que houve um problema. Então, podemos mudar `main` para converter uma variante `Err` em um erro mais prático para nossos usuários, sem o texto ao redor sobre `thread 'main'` e `RUST_BACKTRACE` que uma chamada a `panic!` causa.

A Listagem 12-9 mostra as mudanças que precisamos fazer no valor de retorno da função que agora chamamos de `Config::build` e no corpo da função necessário para retornar um `Result`. Observe que isso não compilará até atualizarmos `main` também, o que faremos na próxima listagem.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("argumentos insuficientes");
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

Fizemos duas mudanças no corpo da função: em vez de chamar `panic!` quando o usuário não passa argumentos suficientes, agora retornamos um valor `Err`; além disso, envolvemos o valor de retorno `Config` em um `Ok`. Essas mudanças fazem a função obedecer à sua nova assinatura de tipo.

Retornar um valor `Err` de `Config::build` permite que a função `main` lide com o valor `Result` retornado pela função `build` e saia do processo de forma mais limpa no caso de erro.

#### Chamando `Config::build` e lidando com erros

Para lidar com o caso de erro e imprimir uma mensagem amigável ao usuário, precisamos atualizar `main` para tratar o `Result` retornado por `Config::build`, como mostrado na Listagem 12-10. Também tiraremos de `panic!` a responsabilidade de encerrar a ferramenta de linha de comando com um código de erro diferente de zero e implementaremos isso manualmente. Um status de saída diferente de zero é uma convenção para sinalizar ao processo que chamou nosso programa que o programa terminou em estado de erro.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Erro ao interpretar argumentos: {err}");
        process::exit(1);
    });

    println!("Buscando {}", config.query);
    println!("No arquivo {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Erro ao ler o arquivo");

    println!("Com o texto:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("argumentos insuficientes");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-10"></a>

[Listagem 12-10](#listagem-12-10): Saindo com código de erro se construir um `Config` falhar

Nesta listagem, usamos um método que ainda não abordamos em detalhes: `unwrap_or_else`, definido em `Result<T, E>` pela biblioteca padrão. Usar `unwrap_or_else` nos permite definir um tratamento de erro personalizado, sem `panic!`. Se o `Result` for um valor `Ok`, o comportamento desse método é semelhante ao de `unwrap`: ele retorna o valor interno que `Ok` envolve. Porém, se o valor for um `Err`, esse método chama o código na closure, que é uma função anônima que definimos e passamos como argumento para `unwrap_or_else`. Abordaremos closures em mais detalhes no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures). Por enquanto, você só precisa saber que `unwrap_or_else` passará o valor interno do `Err`, que neste caso é o literal de string estático `"argumentos insuficientes"` que adicionamos na Listagem 12-9, para nossa closure no argumento `err` que aparece entre as barras verticais. O código na closure pode então usar o valor `err` quando for executado.

Adicionamos uma nova linha `use` para trazer `process` da biblioteca padrão para o escopo. O código na closure que será executado no caso de erro tem apenas duas linhas: imprimimos o valor `err` e então chamamos `process::exit`. A função `process::exit` interromperá o programa imediatamente e retornará o número passado como código de status de saída. Isso é semelhante ao tratamento baseado em `panic!` que usamos na Listagem 12-8, mas não obtemos toda a saída extra. Vamos testar:

```console
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/minigrep`
Erro ao interpretar argumentos: argumentos insuficientes
```

Ótimo! Essa saída é muito mais amigável para nossos usuários.

### Extraindo lógica de `main`

Agora que terminamos de refatorar a análise da configuração, vamos voltar à lógica do programa. Como afirmamos em [“Separando responsabilidades em projetos binários”](#separando-responsabilidades-em-projetos-binarios), vamos extrair uma função chamada `run`, que conterá toda a lógica atualmente na função `main` que não está envolvida em preparar a configuração nem em lidar com erros. Quando terminarmos, a função `main` será concisa e fácil de verificar por inspeção, e poderemos escrever testes para toda a outra lógica.

A Listagem 12-11 mostra a pequena melhoria incremental de extrair uma função `run`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Erro ao interpretar argumentos: {err}");
        process::exit(1);
    });

    println!("Buscando {}", config.query);
    println!("No arquivo {}", config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Erro ao ler o arquivo");

    println!("Com o texto:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("argumentos insuficientes");
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

Com o restante da lógica do programa separado na função `run`, podemos melhorar o tratamento de erros, como fizemos com `Config::build` na Listagem 12-9. Em vez de permitir que o programa entre em pânico chamando `expect`, a função `run` retornará um `Result<T, E>` quando algo der errado. Isso nos permitirá consolidar ainda mais a lógica de tratamento de erros em `main` de uma forma amigável ao usuário. A Listagem 12-12 mostra as mudanças que precisamos fazer na assinatura e no corpo de `run`.

**Arquivo: src/main.rs**

```rust
use std::env;
use std::error::Error;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Erro ao interpretar argumentos: {err}");
        process::exit(1);
    });

    println!("Buscando {}", config.query);
    println!("No arquivo {}", config.file_path);

    run(config);
}

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!("Com o texto:\n{contents}");

    Ok(())
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("argumentos insuficientes");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

<a id="listagem-12-12"></a>

[Listagem 12-12](#listagem-12-12): Mudando a função `run` para retornar `Result`

Fizemos três mudanças significativas aqui. Primeiro, mudamos o tipo de retorno da função `run` para `Result<(), Box<dyn Error>>`. Essa função antes retornava o tipo unitário, `()`, e mantemos isso como o valor retornado no caso `Ok`.

Para o tipo de erro, usamos o trait object `Box<dyn Error>` (e trouxemos `std::error::Error` para o escopo com uma instrução `use` no topo). Abordaremos trait objects no [Capítulo 18](/livro/cap18-00-recursos-de-oop). Por enquanto, saiba apenas que `Box<dyn Error>` significa que a função retornará um tipo que implementa a trait `Error`, mas não precisamos especificar qual tipo concreto será o valor de retorno. Isso nos dá flexibilidade para retornar valores de erro que podem ter tipos diferentes em diferentes casos de erro. A palavra-chave `dyn` é abreviação de _dynamic_.

Segundo, removemos a chamada a `expect` em favor do operador `?`, como falamos no [Capítulo 9](/livro/cap09-00-tratamento-de-erros). Em vez de chamar `panic!` em caso de erro, `?` retornará o valor de erro da função atual para que o chamador lide com ele.

Terceiro, a função `run` agora retorna um valor `Ok` no caso de sucesso. Declaramos o tipo de sucesso da função `run` como `()` na assinatura, o que significa que precisamos envolver o valor do tipo unitário no valor `Ok`. Essa sintaxe `Ok(())` pode parecer um pouco estranha no início. Mas usar `()` assim é a forma idiomática de indicar que estamos chamando `run` apenas por seus efeitos colaterais; ela não retorna um valor de que precisamos.

Quando você executa este código, ele compilará, mas exibirá um aviso:

```console
$ cargo run -- o poema.txt
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

Rust nos diz que nosso código ignorou o valor `Result` e que o valor `Result` pode indicar que ocorreu um erro. Mas não estamos verificando se houve ou não um erro, e o compilador nos lembra que provavelmente queríamos ter algum código de tratamento de erros aqui! Vamos corrigir esse problema agora.

#### Lidando com erros retornados de `run` em `main`

Vamos verificar erros e tratá-los usando uma técnica semelhante à que usamos com `Config::build` na Listagem 12-10, mas com uma pequena diferença:

**Arquivo: src/main.rs**

```rust
    if let Err(e) = run(config) {
        println!("Erro na aplicação: {e}");
        process::exit(1);
    }
```

Usamos `if let` em vez de `unwrap_or_else` para verificar se `run` retorna um valor `Err` e chamar `process::exit(1)` se retornar. A função `run` não retorna um valor que queremos extrair da mesma forma que `Config::build` retorna a instância `Config`. Como `run` retorna `()` no caso de sucesso, só nos importamos em detectar um erro; portanto, não precisamos de `unwrap_or_else` para retornar o valor extraído, que seria apenas `()`.

Os corpos de `if let` e das funções `unwrap_or_else` são os mesmos nos dois casos: imprimimos o erro e saímos.

### Dividindo o código em uma crate de biblioteca

Nosso projeto `minigrep` está indo bem até aqui! Agora vamos dividir o arquivo _src/main.rs_ e colocar parte do código no arquivo _src/lib.rs_. Assim, podemos testar o código e ter um arquivo _src/main.rs_ com menos responsabilidades.

Vamos definir o código responsável por buscar texto em _src/lib.rs_, em vez de em _src/main.rs_, o que permitirá que nós (ou qualquer outra pessoa usando nossa biblioteca `minigrep`) chamemos a função de busca a partir de mais contextos além do nosso binário `minigrep`.

Primeiro, vamos definir a assinatura da função `search` em _src/lib.rs_ como mostrado na Listagem 12-13, com um corpo que chama a macro `unimplemented!`. Explicaremos a assinatura com mais detalhes quando preenchermos a implementação.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    unimplemented!();
}
```

<a id="listagem-12-13"></a>

[Listagem 12-13](#listagem-12-13): Definindo a função `search` em _src/lib.rs_

Usamos a palavra-chave `pub` na definição da função para indicar que `search` faz parte da API pública da nossa crate de biblioteca. Agora temos uma crate de biblioteca que podemos usar a partir da nossa crate binária e que podemos testar!

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

Observe que a função `search` coletará todos os resultados em um vetor que ela retorna antes que qualquer impressão aconteça. Essa implementação pode ser lenta para exibir resultados ao buscar em arquivos grandes, porque os resultados não são impressos conforme são encontrados; discutiremos uma possível forma de corrigir isso usando iterators no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures).

Ufa! Foi bastante trabalho, mas nos preparamos para ter sucesso daqui em diante. Agora ficou muito mais fácil lidar com erros, e tornamos o código mais modular. Quase todo o nosso trabalho será feito em _src/lib.rs_ daqui para frente.

Vamos aproveitar essa nova modularidade fazendo algo que teria sido difícil com o código antigo, mas que é fácil com o novo: escreveremos alguns testes!
