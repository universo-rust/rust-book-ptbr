---
title: "Erros recuperáveis com `Result`"
chapter_code: 09-02
slug: erros-recuperaveis-com-result
---

# Erros Recuperáveis com `Result`

A maioria dos erros não é séria o suficiente para exigir que o programa pare inteiramente. Às vezes, quando uma função falha, é por um motivo que você pode interpretar e responder facilmente. Por exemplo, se você tentar abrir um arquivo e essa operação falhar porque o arquivo não existe, pode querer criar o arquivo em vez de encerrar o processo.

Lembre-se da seção Lidando com falha potencial com `Result` do Capítulo 2 de que o enum `Result` é definido como tendo duas variantes, `Ok` e `Err`, da seguinte forma:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Os `T` e `E` são parâmetros de tipo genérico: discutiremos generics com mais detalhes no Capítulo 10. O que você precisa saber agora é que `T` representa o tipo do valor que será retornado em um caso de sucesso dentro da variante `Ok`, e `E` representa o tipo do erro que será retornado em um caso de falha dentro da variante `Err`. Como `Result` tem esses parâmetros de tipo genérico, podemos usar o tipo `Result` e as funções definidas nele em muitas situações diferentes em que o valor de sucesso e o valor de erro que queremos retornar podem diferir.

Vamos chamar uma função que retorna um valor `Result` porque a função pode falhar. Na Listagem 9-3, tentamos abrir um arquivo.

**Arquivo: src/main.rs**

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

[Listagem 9-3](#listagem-9-3): Abrindo um arquivo

O tipo de retorno de `File::open` é um `Result<T, E>`. O parâmetro genérico `T` foi preenchido pela implementação de `File::open` com o tipo do valor de sucesso, `std::fs::File`, que é um handle de arquivo. O tipo de `E` usado no valor de erro é `std::io::Error`. Este tipo de retorno significa que a chamada a `File::open` pode ter sucesso e retornar um handle de arquivo do qual podemos ler ou escrever. A chamada de função também pode falhar: por exemplo, o arquivo pode não existir, ou podemos não ter permissão para acessar o arquivo. A função `File::open` precisa ter uma forma de nos dizer se teve sucesso ou falhou e, ao mesmo tempo, nos dar o handle de arquivo ou informações de erro. Essa informação é exatamente o que o enum `Result` transmite.

No caso em que `File::open` tem sucesso, o valor na variável `greeting_file_result` será uma instância de `Ok` que contém um handle de arquivo. No caso em que falha, o valor em `greeting_file_result` será uma instância de `Err` que contém mais informações sobre o tipo de erro que ocorreu.

Precisamos adicionar ao código da Listagem 9-3 para tomar ações diferentes dependendo do valor que `File::open` retorna. A Listagem 9-4 mostra uma forma de lidar com o `Result` usando uma ferramenta básica, a expressão `match` que discutimos no Capítulo 6.

**Arquivo: src/main.rs (Este código entra em pânico se `hello.txt` não existir!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

[Listagem 9-4](#listagem-9-4): Usando uma expressão `match` para lidar com as variantes `Result` que podem ser retornadas

Observe que, como o enum `Option`, o enum `Result` e suas variantes foram trazidos para o escopo pelo prelude, então não precisamos especificar `Result::` antes das variantes `Ok` e `Err` nos braços do `match`.

Quando o resultado é `Ok`, este código retornará o valor interno `file` fora da variante `Ok`, e então atribuímos esse valor de handle de arquivo à variável `greeting_file`. Depois do `match`, podemos usar o handle de arquivo para leitura ou escrita.

O outro braço do `match` lida com o caso em que obtemos um valor `Err` de `File::open`. Neste exemplo, escolhemos chamar a macro `panic!`. Se não houver arquivo chamado _hello.txt_ no nosso diretório atual e executarmos este código, veremos a seguinte saída da macro `panic!`:

```bash
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/error-handling`

thread 'main' panicked at src/main.rs:8:23:
Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Como de costume, esta saída nos diz exatamente o que deu errado.

### Fazendo match em erros diferentes

O código da Listagem 9-4 fará `panic!` independentemente do motivo pelo qual `File::open` falhou. Porém, queremos tomar ações diferentes para diferentes motivos de falha. Se `File::open` falhou porque o arquivo não existe, queremos criar o arquivo e retornar o handle do novo arquivo. Se `File::open` falhou por qualquer outro motivo — por exemplo, porque não tínhamos permissão para abrir o arquivo — ainda queremos que o código faça `panic!` da mesma forma que na Listagem 9-4. Para isso, adicionamos uma expressão `match` interna, mostrada na Listagem 9-5.

**Arquivo: src/main.rs**

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            _ => {
                panic!("Problem opening the file: {error:?}");
            }
        },
    };
}
```

[Listagem 9-5](#listagem-9-5): Lidando com diferentes tipos de erros de formas diferentes

O tipo do valor que `File::open` retorna dentro da variante `Err` é `io::Error`, que é uma struct fornecida pela biblioteca padrão. Esta struct tem um método `kind`, que podemos chamar para obter um valor `io::ErrorKind`. O enum `io::ErrorKind` é fornecido pela biblioteca padrão e tem variantes representando os diferentes tipos de erros que podem resultar de uma operação `io`. A variante que queremos usar é `ErrorKind::NotFound`, que indica que o arquivo que estamos tentando abrir ainda não existe. Então, fazemos `match` em `greeting_file_result`, mas também temos um `match` interno em `error.kind()`.

A condição que queremos verificar no `match` interno é se o valor retornado por `error.kind()` é a variante `NotFound` do enum `ErrorKind`. Se for, tentamos criar o arquivo com `File::create`. Porém, como `File::create` também pode falhar, precisamos de um segundo braço na expressão `match` interna. Quando o arquivo não pode ser criado, uma mensagem de erro diferente é impressa. O segundo braço do `match` externo permanece o mesmo, então o programa entra em pânico em qualquer erro além do erro de arquivo ausente.

> #### Alternativas a usar `match` com `Result<T, E>`
>
> Isso é muito `match`! A expressão `match` é muito útil, mas também muito primitiva. No Capítulo 13, você aprenderá sobre closures, que são usadas com muitos dos métodos definidos em `Result<T, E>`. Esses métodos podem ser mais concisos que usar `match` ao lidar com valores `Result<T, E>` no seu código.
>
> Por exemplo, aqui está outra forma de escrever a mesma lógica mostrada na Listagem 9-5, desta vez usando closures e o método `unwrap_or_else`:
>
> ```rust
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> Embora este código tenha o mesmo comportamento da Listagem 9-5, não contém expressões `match` e é mais limpo de ler. Volte a este exemplo depois de ler o Capítulo 13 e consulte o método `unwrap_or_else` na documentação da biblioteca padrão. Muitos outros desses métodos podem limpar enormes expressões `match` aninhadas quando você está lidando com erros.

#### Atalhos para pânico em erro

Usar `match` funciona bem o suficiente, mas pode ser um pouco verboso e nem sempre comunica a intenção bem. O tipo `Result<T, E>` tem muitos métodos auxiliares definidos nele para fazer várias tarefas mais específicas. O método `unwrap` é um método atalho implementado exatamente como a expressão `match` que escrevemos na Listagem 9-4. Se o valor `Result` for a variante `Ok`, `unwrap` retornará o valor dentro do `Ok`. Se o `Result` for a variante `Err`, `unwrap` chamará a macro `panic!` por nós. Aqui está um exemplo de `unwrap` em ação:

**Arquivo: src/main.rs (Este código entra em pânico se `hello.txt` não existir!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

Se executarmos este código sem um arquivo _hello.txt_, veremos uma mensagem de erro da chamada a `panic!` que o método `unwrap` faz:

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Da mesma forma, o método `expect` também nos permite escolher a mensagem de erro do `panic!`. Usar `expect` em vez de `unwrap` e fornecer boas mensagens de erro pode transmitir sua intenção e facilitar rastrear a origem de um pânico. A sintaxe de `expect` se parece com isto:

**Arquivo: src/main.rs (Este código entra em pânico se `hello.txt` não existir!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

Usamos `expect` da mesma forma que `unwrap`: para retornar o handle de arquivo ou chamar a macro `panic!`. A mensagem de erro usada por `expect` em sua chamada a `panic!` será o parâmetro que passamos a `expect`, em vez da mensagem padrão de `panic!` que `unwrap` usa. Veja como fica:

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Em código de qualidade de produção, a maioria dos rustaceans escolhe `expect` em vez de `unwrap` e dá mais contexto sobre por que a operação deve sempre ter sucesso. Assim, se suas suposições forem provadas erradas, você terá mais informação para usar na depuração.

### Propagando erros

Quando a implementação de uma função chama algo que pode falhar, em vez de lidar com o erro dentro da própria função, você pode retornar o erro ao código chamador para que ele decida o que fazer. Isso é conhecido como _propagar_ o erro e dá mais controle ao código chamador, onde pode haver mais informação ou lógica que dita como o erro deve ser tratado do que você tem disponível no contexto do seu código.

Por exemplo, a Listagem 9-6 mostra uma função que lê um nome de usuário de um arquivo. Se o arquivo não existir ou não puder ser lido, esta função retornará esses erros ao código que chamou a função.

**Arquivo: src/main.rs**

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

fn main() {
    let username = read_username_from_file().expect("Unable to get username");
}
```

[Listagem 9-6](#listagem-9-6): Uma função que retorna erros ao código chamador usando `match`

Esta função pode ser escrita de forma muito mais curta, mas vamos começar fazendo muito dela manualmente para explorar o tratamento de erros; no final, mostraremos o caminho mais curto. Vamos olhar primeiro o tipo de retorno da função: `Result<String, io::Error>`. Isso significa que a função retorna um valor do tipo `Result<T, E>`, onde o parâmetro genérico `T` foi preenchido com o tipo concreto `String` e o tipo genérico `E` foi preenchido com o tipo concreto `io::Error`.

Se esta função tiver sucesso sem problemas, o código que chama esta função receberá um valor `Ok` que contém uma `String` — o `username` que esta função leu do arquivo. Se esta função encontrar problemas, o código chamador receberá um valor `Err` que contém uma instância de `io::Error` com mais informações sobre quais foram os problemas. Escolhemos `io::Error` como tipo de retorno desta função porque esse acontece de ser o tipo do valor de erro retornado por ambas as operações que estamos chamando no corpo desta função que podem falhar: a função `File::open` e o método `read_to_string`.

O corpo da função começa chamando a função `File::open`. Em seguida, lidamos com o valor `Result` com um `match` semelhante ao `match` da Listagem 9-4. Se `File::open` tiver sucesso, o handle de arquivo no padrão da variável `file` torna-se o valor na variável mutável `username_file` e a função continua. No caso `Err`, em vez de chamar `panic!`, usamos a palavra-chave `return` para retornar cedo da função inteiramente e passar o valor de erro de `File::open`, agora na variável de padrão `e`, de volta ao código chamador como valor de erro desta função.

Então, se temos um handle de arquivo em `username_file`, a função cria uma nova `String` na variável `username` e chama o método `read_to_string` no handle de arquivo em `username_file` para ler o conteúdo do arquivo em `username`. O método `read_to_string` também retorna um `Result` porque pode falhar, mesmo que `File::open` tenha tido sucesso. Então, precisamos de outro `match` para lidar com esse `Result`: se `read_to_string` tiver sucesso, nossa função teve sucesso e retornamos o username do arquivo que agora está em `username` envolvido em um `Ok`. Se `read_to_string` falhar, retornamos o valor de erro da mesma forma que retornamos o valor de erro no `match` que lidou com o valor de retorno de `File::open`. Porém, não precisamos dizer explicitamente `return`, porque esta é a última expressão na função.

O código que chama este código então lidará com obter um valor `Ok` que contém um username ou um valor `Err` que contém um `io::Error`. Cabe ao código chamador decidir o que fazer com esses valores. Se o código chamador obtiver um valor `Err`, pode chamar `panic!` e fazer o programa entrar em pânico, usar um username padrão ou procurar o username em outro lugar que não seja um arquivo, por exemplo. Não temos informação suficiente sobre o que o código chamador está realmente tentando fazer, então propagamos toda a informação de sucesso ou erro para cima para que ele trate adequadamente.

Este padrão de propagar erros é tão comum em Rust que o Rust fornece o operador ponto de interrogação `?` para facilitar isso.

#### O atalho do operador `?`

A Listagem 9-7 mostra uma implementação de `read_username_from_file` que tem a mesma funcionalidade da Listagem 9-6, mas esta implementação usa o operador `?`.

**Arquivo: src/main.rs**

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}

fn main() {
    let username = read_username_from_file().expect("Unable to get username");
}
```

[Listagem 9-7](#listagem-9-7): Uma função que retorna erros ao código chamador usando o operador `?`

O `?` colocado após um valor `Result` é definido para funcionar de quase a mesma forma que as expressões `match` que definimos para lidar com os valores `Result` na Listagem 9-6. Se o valor do `Result` for um `Ok`, o valor dentro do `Ok` será retornado desta expressão e o programa continuará. Se o valor for um `Err`, o `Err` será retornado de toda a função como se tivéssemos usado a palavra-chave `return`, para que o valor de erro seja propagado ao código chamador.

Há uma diferença entre o que a expressão `match` da Listagem 9-6 faz e o que o operador `?` faz: valores de erro nos quais o operador `?` é chamado passam pela função `from`, definida na trait `From` na biblioteca padrão, que é usada para converter valores de um tipo em outro. Quando o operador `?` chama a função `from`, o tipo de erro recebido é convertido no tipo de erro definido no tipo de retorno da função atual. Isso é útil quando uma função retorna um tipo de erro para representar todas as formas pelas quais uma função pode falhar, mesmo que partes possam falhar por muitos motivos diferentes.

Por exemplo, poderíamos alterar a função `read_username_from_file` na Listagem 9-7 para retornar um tipo de erro personalizado chamado `OurError` que definimos. Se também definirmos `impl From<io::Error> for OurError` para construir uma instância de `OurError` a partir de um `io::Error`, então as chamadas a `?` no corpo de `read_username_from_file` chamarão `from` e converterão os tipos de erro sem precisar adicionar mais código à função.

No contexto da Listagem 9-7, o `?` no final da chamada a `File::open` retornará o valor dentro de um `Ok` para a variável `username_file`. Se ocorrer um erro, o operador `?` retornará cedo de toda a função e dará qualquer valor `Err` ao código chamador. O mesmo se aplica ao `?` no final da chamada a `read_to_string`.

O operador `?` elimina muito boilerplate e torna a implementação desta função mais simples. Podemos até encurtar este código encadeando chamadas de método imediatamente após o `?`, como mostrado na Listagem 9-8.

**Arquivo: src/main.rs**

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}

fn main() {
    let username = read_username_from_file().expect("Unable to get username");
}
```

[Listagem 9-8](#listagem-9-8): Encadeando chamadas de método após o operador `?`

Movemos a criação da nova `String` em `username` para o início da função; essa parte não mudou. Em vez de criar uma variável `username_file`, encadeamos a chamada a `read_to_string` diretamente no resultado de `File::open("hello.txt")?`. Ainda temos um `?` no final da chamada a `read_to_string`, e ainda retornamos um valor `Ok` contendo `username` quando tanto `File::open` quanto `read_to_string` têm sucesso em vez de retornar erros. A funcionalidade é novamente a mesma das Listagens 9-6 e 9-7; esta é apenas uma forma diferente e mais ergonômica de escrevê-la.

A Listagem 9-9 mostra uma forma de tornar isso ainda mais curta usando `fs::read_to_string`.

**Arquivo: src/main.rs**

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}

fn main() {
    let username = read_username_from_file().expect("Unable to get username");
}
```

[Listagem 9-9](#listagem-9-9): Usando `fs::read_to_string` em vez de abrir e depois ler o arquivo

Ler um arquivo em uma string é uma operação bastante comum, então a biblioteca padrão fornece a função conveniente `fs::read_to_string` que abre o arquivo, cria uma nova `String`, lê o conteúdo do arquivo, coloca o conteúdo nessa `String` e a retorna. Claro, usar `fs::read_to_string` não nos dá a oportunidade de explicar todo o tratamento de erros, então fizemos do jeito mais longo primeiro.

#### Onde usar o operador `?`

O operador `?` só pode ser usado em funções cujo tipo de retorno é compatível com o valor no qual o `?` é usado. Isso ocorre porque o operador `?` é definido para realizar um retorno antecipado de um valor da função, da mesma maneira que a expressão `match` que definimos na Listagem 9-6. Na Listagem 9-6, o `match` usava um valor `Result`, e o braço de retorno antecipado retornava um valor `Err(e)`. O tipo de retorno da função precisa ser um `Result` para ser compatível com este `return`.

Na Listagem 9-10, vamos ver o erro que obteremos se usarmos o operador `?` em uma função `main` com um tipo de retorno incompatível com o tipo do valor em que usamos `?`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

[Listagem 9-10](#listagem-9-10): Tentar usar `?` na função `main` que retorna `()` não compilará

Este código abre um arquivo, o que pode falhar. O operador `?` segue o valor `Result` retornado por `File::open`, mas esta função `main` tem o tipo de retorno `()`, não `Result`. Quando compilamos este código, obtemos a seguinte mensagem de erro:

```bash
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
 --> src/main.rs:4:48
  |
3 | fn main() {
  | --------- this function should return `Result` or `Option` to accept `?`
4 |     let greeting_file = File::open("hello.txt")?;
  |                                                ^ cannot use the `?` operator in a function that returns `()`
  |
help: consider adding return type
  |
3 ~ fn main() -> Result<(), Box<dyn std::error::Error>> {
4 |     let greeting_file = File::open("hello.txt")?;
5 +     Ok(())
  |

For more information about this error, try `rustc --explain E0277`.
error: could not compile `error-handling` (bin "error-handling") due to 1 previous error
```

Este erro aponta que só temos permissão para usar o operador `?` em uma função que retorna `Result`, `Option` ou outro tipo que implementa `FromResidual`.

Para corrigir o erro, você tem duas escolhas. Uma é alterar o tipo de retorno da sua função para ser compatível com o valor em que está usando o operador `?`, desde que não tenha restrições que impeçam isso. A outra é usar um `match` ou um dos métodos de `Result<T, E>` para lidar com o `Result<T, E>` da forma apropriada.

A mensagem de erro também mencionou que `?` pode ser usado com valores `Option<T>` também. Como ao usar `?` em `Result`, você só pode usar `?` em `Option` em uma função que retorna um `Option`. O comportamento do operador `?` quando chamado em um `Option<T>` é semelhante ao seu comportamento quando chamado em um `Result<T, E>`: se o valor for `None`, o `None` será retornado cedo da função naquele ponto. Se o valor for `Some`, o valor dentro do `Some` é o valor resultante da expressão, e a função continua. A Listagem 9-11 tem um exemplo de uma função que encontra o último caractere da primeira linha no texto fornecido.

**Arquivo: src/main.rs**

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}

fn main() {
    assert_eq!(
        last_char_of_first_line("Hello, world\nHow are you today?"),
        Some('d')
    );

    assert_eq!(last_char_of_first_line(""), None);
    assert_eq!(last_char_of_first_line("\nhi"), None);
}
```

[Listagem 9-11](#listagem-9-11): Usando o operador `?` em um valor `Option<T>`

Esta função retorna `Option<char>` porque é possível que haja um caractere ali, mas também é possível que não haja. Este código recebe o argumento fatia de string `text` e chama o método `lines` nele, que retorna um iterador sobre as linhas na string. Como esta função quer examinar a primeira linha, chama `next` no iterador para obter o primeiro valor do iterador. Se `text` for a string vazia, esta chamada a `next` retornará `None`, caso em que usamos `?` para parar e retornar `None` de `last_char_of_first_line`. Se `text` não for a string vazia, `next` retornará um valor `Some` contendo uma fatia de string da primeira linha em `text`.

O `?` extrai a fatia de string, e podemos chamar `chars` nessa fatia de string para obter um iterador de seus caracteres. Estamos interessados no último caractere nesta primeira linha, então chamamos `last` para retornar o último item no iterador. Isso é um `Option` porque é possível que a primeira linha seja a string vazia; por exemplo, se `text` começar com uma linha em branco, mas tiver caracteres em outras linhas, como em `"\nhi"`. Porém, se houver um último caractere na primeira linha, ele será retornado na variante `Some`. O operador `?` no meio nos dá uma forma concisa de expressar esta lógica, permitindo implementar a função em uma linha. Se não pudéssemos usar o operador `?` em `Option`, teríamos que implementar esta lógica usando mais chamadas de método ou uma expressão `match`.

Observe que você pode usar o operador `?` em um `Result` em uma função que retorna `Result`, e pode usar o operador `?` em um `Option` em uma função que retorna `Option`, mas não pode misturar. O operador `?` não converterá automaticamente um `Result` em um `Option` ou vice-versa; nesses casos, você pode usar métodos como o método `ok` em `Result` ou o método `ok_or` em `Option` para fazer a conversão explicitamente.

Até agora, todas as funções `main` que usamos retornavam `()`. A função `main` é especial porque é o ponto de entrada e de saída de um programa executável, e há restrições sobre qual pode ser seu tipo de retorno para o programa se comportar como esperado.

Felizmente, `main` também pode retornar um `Result<(), E>`. A Listagem 9-12 tem o código da Listagem 9-10, mas mudamos o tipo de retorno de `main` para `Result<(), Box<dyn Error>>` e adicionamos um valor de retorno `Ok(())` no final. Este código agora compilará.

**Arquivo: src/main.rs**

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

[Listagem 9-12](#listagem-9-12): Alterar `main` para retornar `Result<(), E>` permite o uso do operador `?` em valores `Result`

O tipo `Box<dyn Error>` é um trait object, que falaremos em Usando trait objects para abstrair sobre comportamento compartilhado no Capítulo 18. Por enquanto, você pode ler `Box<dyn Error>` como “qualquer tipo de erro”. Usar `?` em um valor `Result` em uma função `main` com o tipo de erro `Box<dyn Error>` é permitido porque permite que qualquer valor `Err` seja retornado cedo. Embora o corpo desta função `main` só retornará erros do tipo `std::io::Error`, ao especificar `Box<dyn Error>`, esta assinatura continuará correta mesmo se mais código que retorna outros erros for adicionado ao corpo de `main`.

Quando uma função `main` retorna um `Result<(), E>`, o executável sairá com um valor de `0` se `main` retornar `Ok(())` e sairá com um valor diferente de zero se `main` retornar um valor `Err`. Executáveis escritos em C retornam inteiros quando saem: programas que saem com sucesso retornam o inteiro `0`, e programas que erram retornam algum inteiro diferente de `0`. O Rust também retorna inteiros de executáveis para ser compatível com esta convenção.

A função `main` pode retornar quaisquer tipos que implementem a trait `std::process::Termination`, que contém uma função `report` que retorna um `ExitCode`. Consulte a documentação da biblioteca padrão para mais informações sobre implementar a trait `Termination` para seus próprios tipos.

Agora que discutimos os detalhes de chamar `panic!` ou retornar `Result`, vamos voltar ao tópico de como decidir qual é apropriado usar em quais casos.
