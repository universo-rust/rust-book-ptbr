---
title: "Testando a funcionalidade da biblioteca"
chapter_code: 12-04
slug: testando-a-funcionalidade-da-biblioteca
---

# Adicionando funcionalidade com desenvolvimento orientado a testes

Agora que temos a lógica de busca em _src/lib.rs_ separada da função `main`, ficou muito mais fácil escrever testes para a funcionalidade central do nosso código. Podemos chamar funções diretamente com vários argumentos e verificar valores de retorno sem precisar chamar nosso binário pela linha de comando.

Nesta seção, adicionaremos a lógica de busca ao programa `minigrep` usando o processo de desenvolvimento orientado a testes (TDD) com os seguintes passos:

1. Escrever um teste que falha e executá-lo para garantir que falha pelo motivo que você espera.
2. Escrever ou modificar apenas código suficiente para fazer o novo teste passar.
3. Refatorar o código que você acabou de adicionar ou mudar e garantir que os testes continuem passando.
4. Repetir a partir do passo 1!

Embora seja apenas uma entre muitas formas de escrever software, TDD pode ajudar a orientar o design do código. Escrever o teste antes de escrever o código que faz o teste passar ajuda a manter uma alta cobertura de testes ao longo do processo.

Vamos conduzir por TDD a implementação da funcionalidade que de fato fará a busca pela string de consulta no conteúdo do arquivo e produzirá uma lista de linhas que correspondem à consulta. Adicionaremos essa funcionalidade em uma função chamada `search`.

### Escrevendo um teste que falha

Em _src/lib.rs_, adicionaremos um módulo `tests` com uma função de teste, como fizemos no [Capítulo 11](/livro/cap11-00-escrevendo-testes-automatizados). A função de teste especifica o comportamento que queremos que a função `search` tenha: ela receberá uma consulta e o texto no qual buscar, e retornará apenas as linhas do texto que contêm a consulta. A Listagem 12-15 mostra esse teste.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    unimplemented!();
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
}
```

<a id="listagem-12-15"></a>

[Listagem 12-15](#listagem-12-15): Criando um teste que falha para a função `search` com a funcionalidade que desejamos

Este teste busca pela string `"duct"`. O texto em que estamos buscando tem três linhas, das quais apenas uma contém `"duct"` (observe que a barra invertida depois da aspas dupla de abertura diz ao Rust para não colocar um caractere de nova linha no início do conteúdo deste literal de string). Afirmamos que o valor retornado pela função `search` contém apenas a linha que esperamos.

Se executarmos esse teste, ele falhará no momento porque a macro `unimplemented!` entra em pânico com a mensagem “not implemented”. De acordo com os princípios de TDD, daremos um pequeno passo: adicionar apenas código suficiente para que o teste não entre em pânico ao chamar a função, definindo a função `search` para sempre retornar um vetor vazio, como mostrado na Listagem 12-16. Então, o teste deve compilar e falhar porque um vetor vazio não corresponde a um vetor contendo a linha `"safe, fast, productive."`.

**Arquivo: src/lib.rs**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
}
```

<a id="listagem-12-16"></a>

[Listagem 12-16](#listagem-12-16): Definindo apenas o suficiente da função `search` para que chamá-la não entre em pânico

Agora vamos discutir por que precisamos definir um lifetime explícito `'a` na assinatura de `search` e usar esse lifetime com o argumento `contents` e o valor de retorno. Lembre-se, do [Capítulo 10](/livro/cap10-03-validando-referencias-com-lifetimes), de que parâmetros de lifetime especificam qual lifetime de argumento está conectado ao lifetime do valor de retorno. Neste caso, indicamos que o vetor retornado deve conter slices de string que referenciam slices do argumento `contents` (em vez do argumento `query`).

Em outras palavras, dizemos ao Rust que os dados retornados pela função `search` viverão pelo mesmo tempo que os dados passados para a função `search` no argumento `contents`. Isso é importante! Os dados referenciados por um slice precisam ser válidos para que a referência seja válida; se o compilador assumisse que estamos criando slices de string de `query` em vez de `contents`, faria sua verificação de segurança incorretamente.

Se esquecermos as anotações de lifetime e tentarmos compilar esta função, obteremos este erro:

```console
$ cargo build
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
error[E0106]: missing lifetime specifier
 --> src/lib.rs:1:51
  |
1 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
  |                      ----            ----         ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `query` or `contents`
help: consider introducing a named lifetime parameter
  |
1 | pub fn search<'a>(query: &'a str, contents: &'a str) -> Vec<&'a str> {
  |              ++++         ++                 ++              ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `minigrep` (lib) due to 1 previous error
```

Rust não consegue saber qual dos dois parâmetros precisamos para a saída, então precisamos dizer isso explicitamente. Observe que o texto de ajuda sugere especificar o mesmo parâmetro de lifetime para todos os parâmetros e para o tipo de saída, o que está incorreto! Como `contents` é o parâmetro que contém todo o nosso texto e queremos retornar as partes desse texto que correspondem, sabemos que `contents` é o único parâmetro que deve ser conectado ao valor de retorno usando a sintaxe de lifetime.

Outras linguagens de programação não exigem que você conecte argumentos a valores de retorno na assinatura, mas essa prática ficará mais fácil com o tempo. Talvez você queira comparar este exemplo com os exemplos da seção [Validando referências com lifetimes](/livro/cap10-03-validando-referencias-com-lifetimes), no [Capítulo 10](/livro/cap10-00-tipos-genericos-traits-e-lifetimes).

### Escrevendo código para fazer o teste passar

No momento, nosso teste está falhando porque sempre retornamos um vetor vazio. Para corrigir isso e implementar `search`, nosso programa precisa seguir estes passos:

1. Iterar por cada linha de `contents`.
2. Verificar se a linha contém nossa string de consulta.
3. Se contiver, adicioná-la à lista de valores que estamos retornando.
4. Se não contiver, não fazer nada.
5. Retornar a lista de resultados que correspondem.

Vamos passar por cada etapa, começando pela iteração pelas linhas.

#### Iterando pelas linhas com o método `lines`

Rust tem um método útil para lidar com a iteração linha a linha de strings, convenientemente chamado `lines`, que funciona como mostrado na Listagem 12-17. Observe que isso ainda não compilará.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

<a id="listagem-12-17"></a>

[Listagem 12-17](#listagem-12-17): Iterando por cada linha em `contents`

O método `lines` retorna um iterator. Falaremos sobre iterators em profundidade no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures). Mas lembre-se de que você viu essa forma de usar um iterator na Listagem 3-5, onde usamos um loop `for` com um iterator para executar algum código em cada item de uma coleção.

#### Buscando a consulta em cada linha

Em seguida, verificaremos se a linha atual contém nossa string de consulta. Felizmente, strings têm um método útil chamado `contains` que faz isso por nós! Adicione uma chamada ao método `contains` na função `search`, como mostrado na Listagem 12-18. Observe que isso ainda não compilará.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

<a id="listagem-12-18"></a>

[Listagem 12-18](#listagem-12-18): Adicionando funcionalidade para ver se a linha contém a string em `query`

No momento, estamos construindo a funcionalidade. Para que o código compile, precisamos retornar um valor do corpo, como indicamos que faríamos na assinatura da função.

#### Armazenando linhas correspondentes

Para concluir essa função, precisamos de uma forma de armazenar as linhas correspondentes que queremos retornar. Para isso, podemos criar um vetor mutável antes do loop `for` e chamar o método `push` para armazenar uma linha no vetor. Depois do loop `for`, retornamos o vetor, como mostrado na Listagem 12-19.

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

<a id="listagem-12-19"></a>

[Listagem 12-19](#listagem-12-19): Armazenando as linhas que correspondem para podermos retorná-las

Agora a função `search` deve retornar apenas as linhas que contêm `query`, e nosso teste deve passar. Vamos executar o teste:

```console
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 1.22s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 1 test
test tests::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Nosso teste passou, então sabemos que funciona!

Neste ponto, poderíamos considerar oportunidades de refatorar a implementação da função `search`, mantendo os testes passando para preservar a mesma funcionalidade. O código da função `search` não está tão ruim, mas não aproveita alguns recursos úteis de iterators. Voltaremos a este exemplo no [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures), onde exploraremos iterators em detalhes e veremos como melhorá-lo.

Agora o programa inteiro deve funcionar! Vamos testá-lo, primeiro com uma palavra que deve retornar exatamente uma linha do poema de Emily Dickinson: _sapo_.

```console
$ cargo run -- sapo poema.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.38s
     Running `target/debug/minigrep sapo poema.txt`
Que exposto, como um sapo
```

Legal! Agora vamos tentar uma palavra que corresponderá a várias linhas, como _guém_:

```console
$ cargo run -- guém poema.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep guém poema.txt`
Não sou ninguém! Quem é você?
Você também não é ninguém?
Que tedioso ser alguém!
```

E, por fim, vamos garantir que não obtemos nenhuma linha quando buscamos uma palavra que não aparece em lugar nenhum do poema, como _monomorphization_:

```console
$ cargo run -- monomorphization poema.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep monomorphization poema.txt`
```

Excelente! Construímos nossa própria mini versão de uma ferramenta clássica e aprendemos muito sobre como estruturar aplicações. Também aprendemos um pouco sobre entrada e saída de arquivos, lifetimes, testes e análise de linha de comando.

Para completar este projeto, demonstraremos brevemente como trabalhar com variáveis de ambiente e como imprimir no erro padrão, ambos úteis quando você está escrevendo programas de linha de comando.
