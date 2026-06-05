---
title: "Escrevendo em stderr em vez de stdout"
chapter_code: 12-06
slug: escrevendo-em-stderr-em-vez-de-stdout
---

# Redirecionando erros para erro padrão

No momento, estamos escrevendo toda a nossa saída no terminal usando a macro `println!`. Na maioria dos terminais, há dois tipos de saída: _saída padrão_ (`stdout`) para informações gerais e _erro padrão_ (`stderr`) para mensagens de erro. Essa distinção permite que os usuários escolham direcionar a saída bem-sucedida de um programa para um arquivo, mas ainda assim imprimir mensagens de erro na tela.

A macro `println!` só consegue imprimir na saída padrão, então precisamos usar outra coisa para imprimir no erro padrão.

### Verificando onde os erros são escritos

Primeiro, vamos observar como o conteúdo impresso pelo `minigrep` está sendo escrito atualmente na saída padrão, incluindo quaisquer mensagens de erro que queremos escrever no erro padrão. Faremos isso redirecionando o fluxo de saída padrão para um arquivo enquanto causamos intencionalmente um erro. Não redirecionaremos o fluxo de erro padrão, então qualquer conteúdo enviado ao erro padrão continuará sendo exibido na tela.

Espera-se que programas de linha de comando enviem mensagens de erro para o fluxo de erro padrão, para que ainda possamos ver mensagens de erro na tela mesmo se redirecionarmos o fluxo de saída padrão para um arquivo. Nosso programa atualmente não se comporta bem: estamos prestes a ver que ele salva a saída da mensagem de erro em um arquivo!

Para demonstrar esse comportamento, executaremos o programa com `>` e o caminho do arquivo _output.txt_, para o qual queremos redirecionar o fluxo de saída padrão. Não passaremos nenhum argumento, o que deve causar um erro:

```console
$ cargo run > output.txt
```

A sintaxe `>` diz ao shell para escrever o conteúdo da saída padrão em _output.txt_ em vez de escrevê-lo na tela. Não vimos a mensagem de erro esperada impressa na tela, então isso significa que ela deve ter ido parar no arquivo. Isto é o que _output.txt_ contém:

```text
Problem parsing arguments: not enough arguments
```

Sim, nossa mensagem de erro está sendo impressa na saída padrão. É muito mais útil que mensagens de erro como essa sejam impressas no erro padrão, para que apenas os dados de uma execução bem-sucedida acabem no arquivo. Vamos mudar isso.

### Imprimindo erros em erro padrão

Usaremos o código da Listagem 12-24 para mudar a forma como as mensagens de erro são impressas. Por causa da refatoração que fizemos anteriormente neste capítulo, todo o código que imprime mensagens de erro está em uma única função, `main`. A biblioteca padrão fornece a macro `eprintln!`, que imprime no fluxo de erro padrão, então vamos mudar os dois lugares em que chamávamos `println!` para imprimir erros e usar `eprintln!` em vez disso.

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

<a id="listagem-12-24"></a>

[Listagem 12-24](#listagem-12-24): Escrevendo mensagens de erro em erro padrão em vez de saída padrão usando `eprintln!`

Agora vamos executar o programa novamente da mesma forma, sem argumentos e redirecionando a saída padrão com `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Agora vemos o erro na tela, e _output.txt_ não contém nada, que é o comportamento que esperamos de programas de linha de comando.

Vamos executar o programa novamente com argumentos que não causam erro, mas ainda redirecionando a saída padrão para um arquivo, assim:

```console
$ cargo run -- to poem.txt > output.txt
```

Não veremos nenhuma saída no terminal, e _output.txt_ conterá nossos resultados:

**Arquivo: output.txt**

```text
Are you nobody, too?
How dreary to be somebody!
```

Isso demonstra que agora estamos usando a saída padrão para saídas bem-sucedidas e o erro padrão para saídas de erro, conforme apropriado.

## Resumo

Este capítulo recapitulou alguns dos principais conceitos que você aprendeu até agora e abordou como realizar operações comuns de I/O em Rust. Usando argumentos de linha de comando, arquivos, variáveis de ambiente e a macro `eprintln!` para imprimir erros, você agora está preparado para escrever aplicações de linha de comando. Combinado com os conceitos dos capítulos anteriores, seu código será bem organizado, armazenará dados de maneira eficaz nas estruturas de dados apropriadas, lidará bem com erros e será bem testado.

Em seguida, exploraremos alguns recursos do Rust influenciados por linguagens funcionais: closures e iterators.
