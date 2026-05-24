---
title: "Trabalhando com variáveis de ambiente"
chapter_code: 12-05
slug: trabalhando-com-variaveis-de-ambiente
---

# Trabalhando com Variáveis de Ambiente

Melhoraremos o binário `minigrep` adicionando um recurso extra: uma opção de busca que ignora maiúsculas e minúsculas que o usuário pode ativar via variável de ambiente. Poderíamos fazer este recurso uma opção de linha de comando e exigir que o usuário a informe cada vez que quiser que se aplique, mas, ao torná-la uma variável de ambiente, permitimos que nossos usuários definam a variável uma vez e tenham todas as buscas sem distinção de maiúsculas e minúsculas naquela sessão de terminal.

### Escrevendo um teste que falha para busca sem distinção de maiúsculas

Primeiro adicionamos uma nova função `search_case_insensitive` à biblioteca `minigrep` que será chamada quando a variável de ambiente tiver um valor. Continuaremos seguindo o processo TDD, então o primeiro passo é novamente escrever um teste que falha. Adicionaremos um novo teste para a nova função `search_case_insensitive` e renomearemos nosso teste antigo de `one_result` para `case_sensitive` para esclarecer as diferenças entre os dois testes, como mostrado na Listagem 12-20.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

<a id="listagem-12-20"></a>

[Listagem 12-20](#listagem-12-20): Adicionando um novo teste que falha para a função sem distinção de maiúsculas que estamos prestes a adicionar

Observe que também editamos o `contents` do teste antigo. Adicionamos uma nova linha com o texto `"Duct tape."` usando um _D_ maiúsculo que não deve corresponder à consulta `"duct"` quando buscamos de forma sensível a maiúsculas. Mudar o teste antigo assim ajuda a garantir que não quebraremos acidentalmente a funcionalidade de busca sensível a maiúsculas que já implementamos. Este teste deve passar agora e deve continuar passando enquanto trabalhamos na busca sem distinção de maiúsculas.

O novo teste para busca _sem distinção de maiúsculas_ usa `"rUsT"` como consulta. Na função `search_case_insensitive` que estamos prestes a adicionar, a consulta `"rUsT"` deve corresponder à linha contendo `"Rust:"` com _R_ maiúsculo e à linha `"Trust me."` mesmo que ambas tenham capitalização diferente da consulta. Este é nosso teste que falha, e ele falhará ao compilar porque ainda não definimos a função `search_case_insensitive`. Sinta-se à vontade para adicionar uma implementação esqueleto que sempre retorna um vetor vazio, semelhante ao que fizemos para a função `search` na Listagem 12-16, para ver o teste compilar e falhar.

### Implementando a função `search_case_insensitive`

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase a mesma que a função `search`. A única diferença é que colocaremos em minúsculas a `query` e cada `line` para que, qualquer que seja a capitalização dos argumentos de entrada, estejam na mesma capitalização quando verificarmos se a linha contém a consulta.

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

[Listagem 12-21](#listagem-12-21): Definindo a função `search_case_insensitive` para colocar em minúsculas a consulta e a linha antes de compará-las

Primeiro, colocamos em minúsculas a string `query` e a armazenamos em uma nova variável com o mesmo nome, sombreando a `query` original. Chamar `to_lowercase` na consulta é necessário para que, não importa se a consulta do usuário é `"rust"`, `"RUST"`, `"Rust"` ou `"rUsT"`, tratemos a consulta como se fosse `"rust"` e sejamos insensíveis à capitalização. Embora `to_lowercase` lide com Unicode básico, não será 100 por cento preciso. Se estivéssemos escrevendo uma aplicação real, faríamos um pouco mais de trabalho aqui, mas esta seção trata de variáveis de ambiente, não de Unicode, então deixaremos assim aqui.

Observe que `query` agora é uma `String` em vez de um slice de string, porque chamar `to_lowercase` cria novos dados em vez de referenciar dados existentes. Digamos que a consulta seja `"rUsT"`, por exemplo: esse slice de string não contém um `u` ou `t` minúsculos para usarmos, então temos de alocar uma nova `String` contendo `"rust"`. Quando passamos `query` como argumento ao método `contains` agora, precisamos adicionar um e comercial, porque a assinatura de `contains` é definida para receber um slice de string.

Em seguida, adicionamos uma chamada a `to_lowercase` em cada `line` para colocar em minúsculas todos os caracteres. Agora que convertemos `line` e `query` para minúsculas, encontraremos correspondências não importa qual seja a capitalização da consulta.

Vamos ver se esta implementação passa nos testes:

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 1.33s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 2 tests
test tests::case_insensitive ... ok
test tests::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Ótimo! Passaram. Agora vamos chamar a nova função `search_case_insensitive` a partir da função `run`. Primeiro, adicionaremos uma opção de configuração à struct `Config` para alternar entre busca sensível e insensível a maiúsculas. Adicionar este campo causará erros de compilação porque ainda não estamos inicializando este campo em lugar nenhum:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}
```

Em seguida, precisamos que a função `run` verifique o valor do campo `ignore_case` e use isso para decidir se chama a função `search` ou `search_case_insensitive`, como mostrado na Listagem 12-22. Isso ainda não compilará.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
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

<a id="listagem-12-23"></a>

[Listagem 12-23](#listagem-12-23): Verificando se há algum valor na variável de ambiente chamada `IGNORE_CASE`

Aqui, criamos uma nova variável `ignore_case`. Para definir seu valor, chamamos a função `env::var` e passamos o nome da variável de ambiente `IGNORE_CASE`. A função `env::var` retorna um `Result` que será a variante `Ok` bem-sucedida contendo o valor da variável de ambiente se a variável estiver definida com qualquer valor. Retornará a variante `Err` se a variável de ambiente não estiver definida.

Usamos o método `is_ok` no `Result` para verificar se a variável de ambiente está definida, o que significa que o programa deve fazer uma busca sem distinção de maiúsculas. Se a variável de ambiente `IGNORE_CASE` não estiver definida com nada, `is_ok` retornará `false` e o programa fará uma busca sensível a maiúsculas. Não nos importamos com o _valor_ da variável de ambiente, apenas se está definida ou não, então verificamos `is_ok` em vez de usar `unwrap`, `expect` ou qualquer outro método que vimos em `Result`.

Passamos o valor na variável `ignore_case` para a instância de `Config` para que a função `run` possa ler esse valor e decidir se chama `search_case_insensitive` ou `search`, como implementamos na Listagem 12-22.

Vamos tentar! Primeiro, executaremos nosso programa sem a variável de ambiente definida e com a consulta `to`, que deve corresponder a qualquer linha que contenha a palavra _to_ em minúsculas:

```bash
$ cargo run -- to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

Parece que ainda funciona! Agora vamos executar o programa com `IGNORE_CASE` definida como `1`, mas com a mesma consulta `to`:

```bash
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Se você estiver usando PowerShell, precisará definir a variável de ambiente e executar o programa como comandos separados:

```bash
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Isso fará `IGNORE_CASE` persistir pelo restante da sessão do shell. Pode ser removida com o cmdlet `Remove-Item`:

```bash
PS> Remove-Item Env:IGNORE_CASE
```

Devemos obter linhas que contêm _to_ e que podem ter letras maiúsculas:

```bash
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excelente, também obtivemos linhas contendo _To_! Nosso programa `minigrep` agora pode fazer busca sem distinção de maiúsculas controlada por uma variável de ambiente. Agora você sabe como gerenciar opções definidas usando argumentos de linha de comando ou variáveis de ambiente.

Alguns programas permitem argumentos _e_ variáveis de ambiente para a mesma configuração. Nesses casos, os programas decidem que um ou outro tem precedência. Como outro exercício por conta própria, tente controlar sensibilidade a maiúsculas por um argumento de linha de comando ou uma variável de ambiente. Decida se o argumento de linha de comando ou a variável de ambiente deve ter precedência se o programa for executado com um definido para sensível a maiúsculas e outro para ignorar maiúsculas.

O módulo `std::env` contém muitos outros recursos úteis para lidar com variáveis de ambiente: consulte sua documentação para ver o que está disponível.
