---
title: "Erros recuperáveis com `Result`"
chapter_code: 09-02
slug: erros-recuperaveis-com-result
---

# Erros Recuperáveis com `Result`

A maioria dos erros não é grave o bastante para derrubar o programa inteiro. Às vezes uma função falha por um motivo que dá para interpretar e corrigir. Se você tenta abrir um arquivo e a operação falha porque ele não existe, pode preferir criá-lo em vez de encerrar o processo.

Lembre, da seção [Tratando falha potencial com `Result`](/livro/cap02-00-programando-um-jogo-de-adivinhacao#tratando-falha-potencial-com-result) do Capítulo 2, que o enum `Result` tem duas variantes — `Ok` e `Err`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` e `E` são parâmetros de tipo genérico; veremos generics com mais detalhe no Capítulo 10. Por ora, basta saber que `T` é o tipo do valor de sucesso dentro de `Ok`, e `E` é o tipo do erro dentro de `Err`. Com esses parâmetros, `Result` e os métodos definidos nele servem em muitas situações em que sucesso e erro têm tipos diferentes.

Vamos chamar uma função que retorna `Result`, porque ela pode falhar. Na Listagem 9-3, tentamos abrir um arquivo.

**Arquivo: src/main.rs**

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

<a id="listagem-9-3"></a>

[Listagem 9-3](#listagem-9-3): Abrindo um arquivo

O tipo de retorno de `File::open` é `Result<T, E>`. O parâmetro `T` foi preenchido pela implementação com o tipo de sucesso `std::fs::File` — um handle de arquivo. O `E` do erro é `std::io::Error`. Ou seja: a chamada pode ter sucesso e devolver um handle para leitura ou escrita, ou falhar — arquivo inexistente, sem permissão, etc. `File::open` precisa dizer se deu certo ou errado e, ao mesmo tempo, entregar o handle ou informações sobre o erro. Isso é exatamente o que `Result` representa.

Se `File::open` tiver sucesso, `greeting_file_result` será um `Ok` com o handle dentro. Se falhar, será um `Err` com mais detalhes sobre o que aconteceu.

Precisamos estender a Listagem 9-3 para agir de formas distintas conforme o retorno. A Listagem 9-4 usa `match`, ferramenta básica que vimos no Capítulo 6.

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

<a id="listagem-9-4"></a>

[Listagem 9-4](#listagem-9-4): Usando `match` para tratar as variantes de `Result`

Como em `Option`, `Result` e suas variantes entram no escopo pelo prelude — não precisamos escrever `Result::` antes de `Ok` e `Err` nos braços do `match`.

Quando o resultado é `Ok`, o código extrai `file` da variante e atribui a `greeting_file`. Depois do `match`, podemos ler ou escrever no handle.

O outro braço trata `Err` de `File::open`. Aqui escolhemos `panic!`. Sem _hello.txt_ no diretório atual, a saída será:

```console
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/error-handling`

thread 'main' panicked at src/main.rs:8:23:
Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Como de costume, a saída deixa claro o que falhou.

### Tratando erros diferentes

A Listagem 9-4 entra em pânico seja qual for o motivo da falha de `File::open`. Muitas vezes queremos reagir de formas distintas. Se o arquivo não existe, criamos o arquivo e usamos o handle novo. Para qualquer outro erro — sem permissão, por exemplo — mantemos o `panic!` da Listagem 9-4. A Listagem 9-5 adiciona um `match` interno para isso.

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

<a id="listagem-9-5"></a>

[Listagem 9-5](#listagem-9-5): Tratando tipos de erro de formas diferentes

O valor dentro de `Err` em `File::open` é `io::Error`, uma struct da biblioteca padrão. Ela tem o método `kind`, que devolve `io::ErrorKind` — enum com variantes para os erros possíveis em operações de I/O. Queremos `ErrorKind::NotFound`, quando o arquivo ainda não existe. Fazemos `match` em `greeting_file_result` e outro em `error.kind()`.

No `match` interno, checamos se `error.kind()` é `NotFound`. Se for, tentamos `File::create` — que também pode falhar, daí o segundo braço do `match` interno com outra mensagem de erro. O braço externo `_` mantém o pânico para qualquer erro que não seja arquivo ausente.

> #### Alternativas a `match` com `Result<T, E>`
>
> São muitos `match`! `match` é poderoso, mas também bem primitivo. No Capítulo 13 veremos closures, usadas com vários métodos de `Result<T, E>`. Eles podem ser mais concisos que `match` ao lidar com erros.
>
> Outra forma de escrever a mesma lógica da Listagem 9-5, com closures e `unwrap_or_else`:
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
> O comportamento é o da Listagem 9-5, sem nenhum `match` — e mais legível. Volte a este exemplo depois do Capítulo 13 e consulte `unwrap_or_else` na documentação. Há muitos métodos assim para substituir `match` aninhados enormes.

#### Atalhos para pânico em erro

`match` funciona, mas pode ficar verboso e nem sempre deixa a intenção clara. `Result<T, E>` traz métodos auxiliares para tarefas específicas. `unwrap` equivale ao `match` da Listagem 9-4: com `Ok`, devolve o valor interno; com `Err`, chama `panic!` por nós.

**Arquivo: src/main.rs (Este código entra em pânico se `hello.txt` não existir!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

Sem _hello.txt_, a mensagem vem do `panic!` que `unwrap` dispara:

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

`expect` também permite escolher a mensagem de `panic!`. Com mensagens claras, fica mais fácil entender a intenção e rastrear a origem do pânico:

**Arquivo: src/main.rs (Este código entra em pânico se `hello.txt` não existir!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

Usamos `expect` como `unwrap`: devolve o handle ou entra em pânico. A mensagem passada a `expect` substitui a mensagem padrão de `unwrap`:

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Em código de produção, rustaceans costumam preferir `expect` a `unwrap` e dar contexto sobre por que a operação deveria sempre funcionar. Se a suposição falhar, a depuração fica mais fácil.

### Propagando erros

Quando uma função chama algo que pode falhar, em vez de tratar o erro ali, você pode devolvê-lo ao chamador — _propagar_ o erro — para que ele decida o que fazer. O chamador pode ter mais contexto ou lógica do que você dentro da função.

A Listagem 9-6 lê um nome de usuário de um arquivo. Se o arquivo não existir ou não puder ser lido, a função repassa esses erros ao código que a invocou.

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

<a id="listagem-9-6"></a>

[Listagem 9-6](#listagem-9-6): Função que repassa erros ao chamador com `match`

Dá para escrever isso bem mais curto — mas começamos manualmente para explorar o tratamento de erros; no fim mostramos o caminho curto. O retorno é `Result<String, io::Error>`: `T` concreto `String`, `E` concreto `io::Error`.

Em caso de sucesso, o chamador recebe `Ok` com a `String` lida do arquivo. Se algo falhar, recebe `Err` com `io::Error` descrevendo o problema. Escolhemos `io::Error` porque é o tipo de erro de `File::open` e de `read_to_string` — as duas operações que podem falhar aqui.

O corpo chama `File::open` e trata o `Result` com `match`, como na Listagem 9-4. Em `Ok`, o handle vira `username_file` e a função continua. Em `Err`, em vez de `panic!`, usamos `return Err(e)` e devolvemos o erro ao chamador.

Com o handle em `username_file`, criamos `username` e chamamos `read_to_string`. Esse método também retorna `Result` — pode falhar mesmo após `File::open` ter funcionado. Outro `match`: sucesso → `Ok(username)`; falha → `Err(e)`, como no primeiro `match`. Na última expressão, `return` é implícito.

Quem chama decide o que fazer com `Ok` ou `Err`: entrar em pânico, usar um nome padrão, buscar o usuário em outro lugar… Como não sabemos a intenção do chamador, propagamos sucesso e erro para cima.

Esse padrão é tão comum que o Rust oferece o operador `?`.

#### O operador `?`

A Listagem 9-7 faz o mesmo que a 9-6, com `?`.

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

<a id="listagem-9-7"></a>

[Listagem 9-7](#listagem-9-7): Repassando erros ao chamador com o operador `?`

`?` após um `Result` funciona quase como os `match` da Listagem 9-6: em `Ok`, o valor interno segue na expressão; em `Err`, a função retorna cedo e propaga o erro — como se tivéssemos usado `return`.

Há uma diferença: erros com `?` passam por `from`, da trait `From`, que converte tipos. O erro recebido vira o tipo de erro do retorno da função. Isso ajuda quando a função usa um único tipo de erro para representar todas as falhas possíveis.

Por exemplo, poderíamos fazer `read_username_from_file` retornar um `OurError` customizado. Com `impl From<io::Error> for OurError`, os `?` no corpo chamariam `from` e converteriam os tipos sem código extra.

Na Listagem 9-7, `?` em `File::open` coloca o valor de `Ok` em `username_file`; em erro, sai da função e repassa `Err`. O mesmo vale para `?` em `read_to_string`.

`?` elimina boilerplate e simplifica a função. Dá para encurtar ainda mais encadeando métodos após `?`, como na Listagem 9-8.

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

<a id="listagem-9-8"></a>

[Listagem 9-8](#listagem-9-8): Encadeando métodos após `?`

A criação de `username` no início permanece. Em vez de `username_file`, encadeamos `read_to_string` no resultado de `File::open("hello.txt")?`. O `?` final em `read_to_string` e o `Ok(username)` no sucesso continuam iguais. Mesma funcionalidade das Listagens 9-6 e 9-7 — só que mais ergonômica.

A Listagem 9-9 encurta ainda mais com `fs::read_to_string`.

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

<a id="listagem-9-9"></a>

[Listagem 9-9](#listagem-9-9): Usando `fs::read_to_string` em vez de abrir e ler

Ler arquivo em string é comum; a biblioteca padrão oferece `fs::read_to_string`, que abre, cria a `String`, lê o conteúdo, coloca na string e devolve. Com ela não dá para percorrer cada passo do tratamento de erros — por isso fizemos o caminho longo primeiro.

#### Onde usar `?`

`?` só funciona em funções cujo tipo de retorno é compatível com o valor em que `?` é aplicado. Ele faz retorno antecipado, como o `match` da Listagem 9-6 — cujo braço de saída devolvia `Err(e)`. O retorno precisa ser `Result` (ou compatível).

Na Listagem 9-10, vemos o erro ao usar `?` em `main` com retorno `()`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

<a id="listagem-9-10"></a>

[Listagem 9-10](#listagem-9-10): Usar `?` em `main` que retorna `()` não compila

Abrir o arquivo pode falhar. `?` segue o `Result` de `File::open`, mas `main` retorna `()`, não `Result`. Ao compilar:

```console
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

Só é permitido usar `?` em funções que retornam `Result`, `Option` ou outro tipo que implementa `FromResidual`.

Para corrigir: alterar o retorno da função para ser compatível com `?` (se não houver outra restrição), ou tratar o `Result` com `match` ou métodos de `Result`.

A mensagem também diz que `?` funciona com `Option<T>`. Como em `Result`, só em função que retorna `Option`. Com `Option<T>`, se for `None`, a função retorna `None` na hora; se for `Some`, o valor interno segue e a função continua. A Listagem 9-11 encontra o último caractere da primeira linha de um texto.

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

<a id="listagem-9-11"></a>

[Listagem 9-11](#listagem-9-11): Usando `?` em um valor `Option<T>`

Retorna `Option<char>` porque pode haver caractere — ou não. `lines()` devolve um iterador sobre as linhas; `next()` pega a primeira. Se `text` for vazio, `next()` dá `None` e `?` encerra a função retornando `None`. Caso contrário, `next()` dá `Some` com a fatia da primeira linha.

`?` extrai a fatia; `chars()` itera os caracteres; `last()` pega o último — também `Option`, porque a primeira linha pode ser vazia (ex.: `"\nhi"`). Se houver último caractere, vem em `Some`. O `?` no meio condensa a lógica numa linha. Sem ele, seriam mais métodos ou um `match`.

`?` em `Result` exige função que retorna `Result`; em `Option`, função que retorna `Option` — não dá para misturar. `?` não converte `Result` em `Option` automaticamente; use `ok` em `Result` ou `ok_or` em `Option` quando precisar converter.

Até aqui, todas as funções `main` retornavam `()`. `main` é especial: ponto de entrada e saída do executável, com restrições no tipo de retorno.

Felizmente, `main` também pode retornar `Result<(), E>`. A Listagem 9-12 repete a 9-10, mas com `main` retornando `Result<(), Box<dyn Error>>` e `Ok(())` no fim. Agora compila.

**Arquivo: src/main.rs**

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

<a id="listagem-9-12"></a>

[Listagem 9-12](#listagem-9-12): `main` retornando `Result<(), E>` permite `?` em valores `Result`

`Box<dyn Error>` é um trait object — veremos em [Usando trait objects para abstrair comportamento compartilhado](/livro/cap18-02-usando-trait-objects-para-abstrair-comportamento-compartilhado) no Capítulo 18. Por ora, leia como “qualquer tipo de erro”. Com `Box<dyn Error>`, qualquer `Err` pode sair cedo de `main`. Mesmo que hoje só haja `std::io::Error`, a assinatura continua válida se você adicionar outros erros depois.

Quando `main` retorna `Result<(), E>`, o executável sai com código `0` em `Ok(())` e com valor diferente de zero em `Err`. Em C, programas bem-sucedidos retornam `0` e falhas retornam outro inteiro. Rust segue a mesma convenção.

`main` também pode retornar tipos que implementam `std::process::Termination`, com `report` devolvendo `ExitCode`. Consulte a documentação para implementar `Termination` nos seus tipos.

Agora que vimos `panic!` e `Result` em detalhe, voltamos à pergunta: quando usar cada um?
