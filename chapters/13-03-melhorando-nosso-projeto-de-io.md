---
title: "Melhorando nosso projeto de I/O"
chapter_code: 13-03
slug: melhorando-nosso-projeto-de-io
---

# Melhorando Nosso Projeto de I/O

Com este novo conhecimento sobre iterators, podemos melhorar o projeto de I/O do Capítulo 12 usando iterators para tornar partes do código mais claras e concisas. Vamos ver como iterators podem melhorar nossa implementação da função `Config::build` e da função `search`.

## Removendo um `clone` Usando um Iterator

Na Listagem 12-6, adicionamos código que pegava um slice de valores `String` e criava uma instância da struct `Config` indexando o slice e clonando os valores, permitindo que a struct `Config` possua esses valores. Na Listagem 13-17, reproduzimos a implementação da função `Config::build` como estava na Listagem 12-23.

**Arquivo: src/main.rs**

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

<a id="listagem-13-17"></a>

[Listagem 13-17](#listagem-13-17): Reprodução da função `Config::build` da Listagem 12-23

Na época, dissemos para não se preocupar com as chamadas ineficientes a `clone` porque as removeríamos no futuro. Bem, esse momento chegou!

Precisávamos de `clone` aqui porque temos um slice com elementos `String` no parâmetro `args`, mas a função `build` não possui `args`. Para retornar posse de uma instância de `Config`, tivemos que clonar os valores dos campos `query` e `file_path` de `Config` para que a instância de `Config` possa possuir seus valores.

Com nosso novo conhecimento sobre iterators, podemos mudar a função `build` para receber posse de um iterator como argumento em vez de emprestar um slice. Usaremos a funcionalidade do iterator em vez do código que verifica o comprimento do slice e indexa posições específicas. Isso deixará mais claro o que a função `Config::build` está fazendo, porque o iterator acessará os valores.

Quando `Config::build` receber posse do iterator e parar de usar operações de indexação que emprestam, poderemos mover os valores `String` do iterator para `Config` em vez de chamar `clone` e fazer uma nova alocação.

### Usando o Iterator Retornado Diretamente

Abra o arquivo _src/main.rs_ do seu projeto de I/O, que deve se parecer com isto:

**Arquivo: src/main.rs**

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

Primeiro mudaremos o início da função `main` que tínhamos na Listagem 12-24 para o código da Listagem 13-18, que desta vez usa um iterator. Isso não compilará até atualizarmos `Config::build` também.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

<a id="listagem-13-18"></a>

[Listagem 13-18](#listagem-13-18): Passando o valor de retorno de `env::args` para `Config::build`

A função `env::args` retorna um iterator! Em vez de coletar os valores do iterator em um vetor e então passar um slice para `Config::build`, agora estamos passando diretamente a posse do iterator retornado por `env::args` para `Config::build`.

Em seguida, precisamos atualizar a definição de `Config::build`. Vamos mudar a assinatura de `Config::build` para parecer com a Listagem 13-19. Isso ainda não compilará, porque precisamos atualizar o corpo da função.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
impl Config {
    fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

<a id="listagem-13-19"></a>

[Listagem 13-19](#listagem-13-19): Atualizando a assinatura de `Config::build` para esperar um iterator

A documentação da biblioteca padrão para a função `env::args` mostra que o tipo do iterator que ela retorna é `std::env::Args`, e esse tipo implementa a trait `Iterator` e retorna valores `String`.

Atualizamos a assinatura da função `Config::build` para que o parâmetro `args` tenha um tipo genérico com o trait bound `impl Iterator<Item = String>` em vez de `&[String]`. Este uso da sintaxe `impl Trait` que discutimos na seção Usando traits como parâmetros do Capítulo 10 significa que `args` pode ser qualquer tipo que implemente a trait `Iterator` e retorne itens `String`.

Como estamos tomando posse de `args` e vamos mutar `args` iterando sobre ele, podemos adicionar a palavra-chave `mut` na especificação do parâmetro `args` para torná-lo mutável.

### Usando Métodos da Trait `Iterator`

Em seguida, corrigiremos o corpo de `Config::build`. Como `args` implementa a trait `Iterator`, sabemos que podemos chamar o método `next` nele! A Listagem 13-20 atualiza o código da Listagem 12-23 para usar o método `next`.

**Arquivo: src/main.rs**

```rust
impl Config {
    fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

<a id="listagem-13-20"></a>

[Listagem 13-20](#listagem-13-20): Mudando o corpo de `Config::build` para usar métodos de iterator

Lembre-se de que o primeiro valor no valor de retorno de `env::args` é o nome do programa. Queremos ignorar isso e ir para o próximo valor, então primeiro chamamos `next` e não fazemos nada com o valor de retorno. Depois, chamamos `next` para obter o valor que queremos colocar no campo `query` de `Config`. Se `next` retornar `Some`, usamos um `match` para extrair o valor. Se retornar `None`, significa que não foram dados argumentos suficientes, e retornamos cedo com um valor `Err`. Fazemos a mesma coisa para o valor `file_path`.

## Deixando o Código Mais Claro com Adapters de Iterator

Também podemos aproveitar iterators na função `search` do nosso projeto de I/O, reproduzida aqui na Listagem 13-21 como estava na Listagem 12-19.

**Arquivo: src/lib.rs**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<a id="listagem-13-21"></a>

[Listagem 13-21](#listagem-13-21): A implementação da função `search` da Listagem 12-19

Podemos escrever este código de forma mais concisa usando métodos adapter de iterator. Isso também nos permite evitar ter um vetor intermediário mutável `results`. O estilo de programação funcional prefere minimizar a quantidade de estado mutável para deixar o código mais claro. Remover o estado mutável pode permitir uma melhoria futura para fazer a busca em paralelo, porque não precisaríamos gerenciar acesso concorrente ao vetor `results`. A Listagem 13-22 mostra essa mudança.

**Arquivo: src/lib.rs**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

<a id="listagem-13-22"></a>

[Listagem 13-22](#listagem-13-22): Usando métodos adapter de iterator na implementação da função `search`

Lembre-se de que o propósito da função `search` é retornar todas as linhas em `contents` que contêm `query`. Semelhante ao exemplo com `filter` na Listagem 13-16, este código usa o adapter `filter` para manter apenas as linhas para as quais `line.contains(query)` retorna `true`. Em seguida, coletamos as linhas correspondentes em outro vetor com `collect`. Muito mais simples! Sinta-se à vontade para fazer a mesma mudança para usar métodos de iterator na função `search_case_insensitive` também.

Para uma melhoria adicional, retorne um iterator da função `search` removendo a chamada a `collect` e mudando o tipo de retorno para `impl Iterator<Item = &'a str>` para que a função se torne um adapter de iterator. Note que você também precisará atualizar os testes! Pesquise em um arquivo grande usando sua ferramenta `minigrep` antes e depois de fazer essa mudança para observar a diferença de comportamento. Antes desta mudança, o programa não imprimirá nenhum resultado até ter coletado todos os resultados, mas depois da mudança, os resultados serão impressos conforme cada linha correspondente for encontrada, porque o loop `for` na função `run` pode aproveitar a preguiça do iterator.

## Escolhendo entre Loops e Iterators

A próxima pergunta lógica é qual estilo você deve escolher no seu próprio código e por quê: a implementação original na Listagem 13-21 ou a versão que usa iterators na Listagem 13-22 (assumindo que estamos coletando todos os resultados antes de retorná-los em vez de retornar o iterator). A maioria dos programadores Rust prefere o estilo com iterator. É um pouco mais difícil de pegar o jeito no começo, mas depois que você se acostuma com os vários adapters de iterator e o que eles fazem, iterators podem ser mais fáceis de entender. Em vez de mexer com os vários pedaços de loop e construção de novos vetores, o código foca no objetivo de alto nível do loop. Isso abstrai parte do código comum para que seja mais fácil ver os conceitos únicos deste código, como a condição de filtragem que cada elemento do iterator deve passar.

Mas as duas implementações são realmente equivalentes? A suposição intuitiva pode ser que o loop de nível mais baixo será mais rápido. Vamos falar sobre desempenho.
