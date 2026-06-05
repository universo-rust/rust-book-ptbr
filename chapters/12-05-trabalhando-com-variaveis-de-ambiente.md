---
title: "Trabalhando com variáveis de ambiente"
chapter_code: 12-05
slug: trabalhando-com-variaveis-de-ambiente
---

# Trabalhando com variáveis de ambiente

Vamos melhorar o binário `minigrep` adicionando um recurso extra: uma opção de busca sem diferenciar maiúsculas de minúsculas que o usuário pode ativar por meio de uma variável de ambiente. Poderíamos transformar esse recurso em uma opção de linha de comando e exigir que os usuários a informassem toda vez que quisessem aplicá-la, mas, ao torná-la uma variável de ambiente, permitimos que nossos usuários definam a variável uma vez e façam todas as buscas sem diferenciar maiúsculas de minúsculas naquela sessão de terminal.

### Escrevendo um teste que falha para busca sem diferenciar maiúsculas de minúsculas

Primeiro, adicionamos uma nova função `search_case_insensitive` à biblioteca `minigrep`, que será chamada quando a variável de ambiente tiver um valor. Continuaremos seguindo o processo TDD, então o primeiro passo é novamente escrever um teste que falha. Adicionaremos um novo teste para a nova função `search_case_insensitive` e renomearemos nosso teste antigo de `one_result` para `case_sensitive`, para esclarecer as diferenças entre os dois testes, como mostrado na Listagem 12-20.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "seguro";
        let contents = "\
Rust:
seguro, rápido, produtivo.
Escolha três.
Fita adesiva.";

        assert_eq!(vec!["seguro, rápido, produtivo."], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = "ráPIdo";
        let contents = "\
Rust:
seguro, rápido, produtivo.
Escolha três.
Trabalho RÁPIDO.";

        assert_eq!(
            vec!["seguro, rápido, produtivo.", "Trabalho RÁPIDO."],
            search_case_insensitive(query, contents)
        );
    }
}
```

<a id="listagem-12-20"></a>

[Listagem 12-20](#listagem-12-20): Adicionando um novo teste que falha para a função sem diferenciar maiúsculas de minúsculas que estamos prestes a adicionar

Observe que também editamos o `contents` do teste antigo. Adicionamos uma nova linha com o texto `"Fita adesiva."`, que não contém a consulta `"seguro"`. Alterar o teste antigo dessa forma ajuda a garantir que não quebremos acidentalmente a funcionalidade de busca sensível a maiúsculas e minúsculas que já implementamos. Esse teste deve passar agora e deve continuar passando enquanto trabalhamos na busca sem diferenciar maiúsculas de minúsculas.

O novo teste para busca _sem diferenciar maiúsculas de minúsculas_ usa `"ráPIdo"` como consulta. Na função `search_case_insensitive` que estamos prestes a adicionar, a consulta `"ráPIdo"` deve corresponder à linha que contém `"rápido"` com capitalização diferente e também à linha `"Trabalho RÁPIDO."`, embora ambas tenham capitalização diferente da consulta. Esse é nosso teste que falha, e ele falhará ao compilar porque ainda não definimos a função `search_case_insensitive`. Sinta-se à vontade para adicionar uma implementação esqueleto que sempre retorna um vetor vazio, semelhante ao que fizemos para a função `search` na Listagem 12-16, para ver o teste compilar e falhar.

### Implementando a função `search_case_insensitive`

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase igual à função `search`. A única diferença é que converteremos a consulta e cada linha para minúsculas, de modo que, qualquer que seja a capitalização dos argumentos de entrada, eles estarão na mesma capitalização quando verificarmos se a linha contém a consulta.

**Arquivo: src/lib.rs**

```rust
pub fn search_case_insensitive<'a>(
    query: &str,
    contents: &'a str,
) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

<a id="listagem-12-21"></a>

[Listagem 12-21](#listagem-12-21): Definindo a função `search_case_insensitive` para converter a consulta e a linha para minúsculas antes de compará-las

Primeiro, convertemos a string `query` para minúsculas e a armazenamos em uma nova variável com o mesmo nome, sombreando a `query` original. Chamar `to_lowercase` na consulta é necessário para que, independentemente de a consulta do usuário ser `"rápido"`, `"RÁPIDO"`, `"Rápido"` ou `"ráPIdo"`, tratemos a consulta como se fosse `"rápido"` e ignoremos a diferença entre maiúsculas e minúsculas. Embora `to_lowercase` lide com Unicode básico, ele não será 100 por cento preciso. Se estivéssemos escrevendo uma aplicação real, faríamos um pouco mais de trabalho aqui, mas esta seção trata de variáveis de ambiente, não de Unicode, então vamos deixar assim.

Observe que `query` agora é uma `String`, em vez de um slice de string, porque chamar `to_lowercase` cria novos dados em vez de referenciar dados existentes. Digamos que a consulta seja `"ráPIdo"`, por exemplo: esse slice de string não contém um `i` nem um `d` minúsculos para usarmos, então precisamos alocar uma nova `String` contendo `"rápido"`. Quando passamos `query` como argumento para o método `contains` agora, precisamos adicionar um e comercial, porque a assinatura de `contains` é definida para receber um slice de string.

Em seguida, adicionamos uma chamada a `to_lowercase` em cada `line` para converter todos os caracteres para minúsculas. Agora que convertemos `line` e `query` para minúsculas, encontraremos correspondências independentemente da capitalização da consulta.

Vamos ver se esta implementação passa nos testes:

```console
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 1.33s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 2 tests
test tests::case_insensitive ... ok
test tests::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Ótimo! Eles passaram. Agora vamos chamar a nova função `search_case_insensitive` a partir da função `run`. Primeiro, adicionaremos uma opção de configuração à struct `Config` para alternar entre busca sensível e insensível a maiúsculas e minúsculas. Adicionar esse campo causará erros de compilação porque ainda não o estamos inicializando em lugar nenhum:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}
```

Adicionamos o campo `ignore_case`, que armazena um booleano. Em seguida, precisamos que a função `run` verifique o valor do campo `ignore_case` e use isso para decidir se chama a função `search` ou a função `search_case_insensitive`, como mostrado na Listagem 12-22. Isso ainda não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use minigrep::{search, search_case_insensitive};

// --snip--

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}
```

<a id="listagem-12-22"></a>

[Listagem 12-22](#listagem-12-22): Chamando `search` ou `search_case_insensitive` com base no valor em `config.ignore_case`

Por fim, precisamos verificar a variável de ambiente. As funções para trabalhar com variáveis de ambiente estão no módulo `env` da biblioteca padrão, que já está no escopo no topo de _src/main.rs_. Usaremos a função `var` do módulo `env` para verificar se algum valor foi definido para uma variável de ambiente chamada `IGNORE_CASE`, como mostrado na Listagem 12-23.

**Arquivo: src/main.rs**

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("argumentos insuficientes");
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

<a id="listagem-12-23"></a>

[Listagem 12-23](#listagem-12-23): Verificando se há algum valor na variável de ambiente chamada `IGNORE_CASE`

Aqui, criamos uma nova variável, `ignore_case`. Para definir seu valor, chamamos a função `env::var` e passamos o nome da variável de ambiente `IGNORE_CASE`. A função `env::var` retorna um `Result` que será a variante bem-sucedida `Ok`, contendo o valor da variável de ambiente, se a variável estiver definida com qualquer valor. Ela retornará a variante `Err` se a variável de ambiente não estiver definida.

Usamos o método `is_ok` no `Result` para verificar se a variável de ambiente está definida, o que significa que o programa deve fazer uma busca sem diferenciar maiúsculas de minúsculas. Se a variável de ambiente `IGNORE_CASE` não estiver definida com nada, `is_ok` retornará `false` e o programa fará uma busca sensível a maiúsculas e minúsculas. Não nos importamos com o _valor_ da variável de ambiente, apenas se ela está definida ou não, então verificamos `is_ok` em vez de usar `unwrap`, `expect` ou qualquer outro método que vimos em `Result`.

Passamos o valor na variável `ignore_case` para a instância de `Config` para que a função `run` possa ler esse valor e decidir se chama `search_case_insensitive` ou `search`, como implementamos na Listagem 12-22.

Vamos tentar! Primeiro, executaremos nosso programa sem a variável de ambiente definida e com a consulta `te`, que deve corresponder a qualquer linha que contenha essas letras em sequência:

```console
$ cargo run -- te poema.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep te poema.txt`
Então somos um par — não conte!
Que tedioso ser alguém!
Dizer o seu nome o dia inteiro
```

Parece que ainda funciona! Agora vamos executar o programa com `IGNORE_CASE` definida como `1`, mas com a mesma consulta `te`:

```console
$ IGNORE_CASE=1 cargo run -- te poema.txt
```

Se você estiver usando PowerShell, precisará definir a variável de ambiente e executar o programa como comandos separados:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- te poema.txt
```

Isso fará `IGNORE_CASE` persistir pelo restante da sessão do shell. Pode ser removida com o cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Devemos obter as mesmas linhas — neste poema, todas as ocorrências de `te` já estão em minúsculas. Se o texto tivesse `Te` ou `TE`, elas também seriam encontradas com `IGNORE_CASE` definida:

```console
Então somos um par — não conte!
Que tedioso ser alguém!
Dizer o seu nome o dia inteiro
```

Excelente! Nosso programa `minigrep` agora pode fazer buscas sem diferenciar maiúsculas de minúsculas, controladas por uma variável de ambiente. Agora você sabe como gerenciar opções definidas usando argumentos de linha de comando ou variáveis de ambiente.

Alguns programas permitem argumentos _e_ variáveis de ambiente para a mesma configuração. Nesses casos, os programas decidem que um ou outro tem precedência. Como outro exercício por conta própria, tente controlar a sensibilidade a maiúsculas e minúsculas por meio de um argumento de linha de comando ou de uma variável de ambiente. Decida se o argumento de linha de comando ou a variável de ambiente deve ter precedência se o programa for executado com uma opção definida para ser sensível a maiúsculas e minúsculas e outra definida para ignorar essa diferença.

O módulo `std::env` contém muitos outros recursos úteis para lidar com variáveis de ambiente: consulte sua documentação para ver o que está disponível.
