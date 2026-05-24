---
title: "Servidor multithread"
chapter_code: 21-02
slug: servidor-multithread
---

# Servidor Multithread

<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## De um servidor de uma única thread a um servidor multithread

No momento, o servidor processará cada requisição em sequência, o que significa que não processará uma segunda conexão até que a primeira conexão termine de ser processada. Se o servidor receber cada vez mais requisições, essa execução serial será cada vez menos ideal. Se o servidor receber uma requisição que demora muito para processar, as requisições subsequentes terão de esperar até que a requisição longa termine, mesmo que as novas requisições possam ser processadas rapidamente. Precisamos corrigir isso, mas primeiro veremos o problema em ação.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Simulando uma requisição lenta

Veremos como uma requisição que processa lentamente pode afetar outras requisições feitas à implementação atual do nosso servidor. A Listagem 21-10 implementa o tratamento de uma requisição para _/sleep_ com uma resposta lenta simulada que fará o servidor dormir por cinco segundos antes de responder.

**Arquivo: src/main.rs**

```rust,no_run
use std::{
    fs,
    io::{BufReader, prelude::*},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};
// --snip--

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    // --snip--

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

<a id="listagem-21-10"></a>

[Listagem 21-10](#listagem-21-10): Simulando uma requisição lenta dormindo por cinco segundos

Mudamos de `if` para `match` agora que temos três casos. Precisamos casar explicitamente com um slice de `request_line` para fazer pattern matching contra os valores literais de string; `match` não faz referenciamento e desreferenciamento automáticos, como o método de igualdade faz.

O primeiro braço é o mesmo que o bloco `if` da Listagem 21-9. O segundo braço casa com uma requisição para _/sleep_. Quando essa requisição é recebida, o servidor dormirá por cinco segundos antes de renderizar a página HTML de sucesso. O terceiro braço é o mesmo que o bloco `else` da Listagem 21-9.

Você pode ver como nosso servidor é primitivo: bibliotecas reais tratarão o reconhecimento de várias requisições de forma muito menos verbosa!

Inicie o servidor com `cargo run`. Em seguida, abra duas janelas do navegador: uma para _http://127.0.0.1:7878_ e outra para _http://127.0.0.1:7878/sleep_. Se você acessar a URI _/_ algumas vezes, como antes, verá que ela responde rapidamente. Mas se você acessar _/sleep_ e depois carregar _/_, verá que _/_ espera até que `sleep` tenha dormido os cinco segundos completos antes de carregar.

Existem várias técnicas que poderíamos usar para evitar que requisições fiquem enfileiradas atrás de uma requisição lenta, incluindo o uso de async como fizemos no Capítulo 17; a que implementaremos é um pool de threads.

### Melhorando a taxa de transferência com um pool de threads

Um _pool de threads_ é um grupo de threads criadas que estão prontas e esperando para lidar com uma tarefa. Quando o programa recebe uma nova tarefa, atribui uma das threads do pool à tarefa, e essa thread processará a tarefa. As threads restantes no pool estão disponíveis para lidar com quaisquer outras tarefas que cheguem enquanto a primeira thread está processando. Quando a primeira thread termina de processar sua tarefa, ela é devolvida ao pool de threads ociosas, pronta para lidar com uma nova tarefa. Um pool de threads permite processar conexões de forma concorrente, aumentando a taxa de transferência do seu servidor.

Limitaremos o número de threads no pool a um número pequeno para nos proteger de ataques DoS; se nosso programa criasse uma nova thread para cada requisição à medida que chegasse, alguém fazendo 10 milhões de requisições ao nosso servidor poderia causar estragos ao consumir todos os recursos do servidor e paralisar o processamento de requisições.

Em vez de criar threads ilimitadas, então, teremos um número fixo de threads esperando no pool. As requisições que chegam são enviadas ao pool para processamento. O pool manterá uma fila de requisições recebidas. Cada uma das threads no pool retirará uma requisição dessa fila, tratará a requisição e então pedirá outra requisição à fila. Com esse design, podemos processar até _`N`_ requisições de forma concorrente, onde _`N`_ é o número de threads. Se cada thread estiver respondendo a uma requisição de longa duração, as requisições subsequentes ainda podem se acumular na fila, mas aumentamos o número de requisições de longa duração que podemos lidar antes de chegar a esse ponto.

Essa técnica é apenas uma entre muitas formas de melhorar a taxa de transferência de um servidor web. Outras opções que você pode explorar são o modelo fork/join, o modelo de I/O assíncrono de uma única thread e o modelo de I/O assíncrono multithread. Se você se interessar por esse tema, pode ler mais sobre outras soluções e tentar implementá-las; com uma linguagem de baixo nível como Rust, todas essas opções são possíveis.

Antes de começarmos a implementar um pool de threads, vamos falar sobre como o uso do pool deve parecer. Quando você está tentando projetar código, escrever primeiro a interface do cliente pode ajudar a orientar seu design. Escreva a API do código de forma que ela seja estruturada da maneira como você quer chamá-la; então, implemente a funcionalidade dentro dessa estrutura, em vez de implementar a funcionalidade e depois projetar a API pública.

De forma semelhante a como usamos desenvolvimento orientado a testes no projeto do Capítulo 12, usaremos aqui desenvolvimento orientado pelo compilador. Escreveremos o código que chama as funções que queremos e então olharemos os erros do compilador para determinar o que devemos mudar em seguida para fazer o código funcionar. Antes de fazermos isso, porém, exploraremos a técnica que não vamos usar como ponto de partida.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Criando uma thread para cada requisição

Primeiro, vamos explorar como nosso código poderia parecer se criasse uma nova thread para cada conexão. Como mencionado anteriormente, este não é nosso plano final devido aos problemas de potencialmente criar um número ilimitado de threads, mas é um ponto de partida para obter primeiro um servidor multithread funcional. Depois, adicionaremos o pool de threads como melhoria, e contrastar as duas soluções será mais fácil.

A Listagem 21-11 mostra as mudanças a fazer em `main` para criar uma nova thread para tratar cada stream dentro do loop `for`.

**Arquivo: src/main.rs**

```rust,no_run
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

<a id="listagem-21-11"></a>

[Listagem 21-11](#listagem-21-11): Criando uma nova thread para cada stream

Como você aprendeu no Capítulo 16, `thread::spawn` criará uma nova thread e então executará o código no closure na nova thread. Se você executar este código e carregar _/sleep_ no navegador, depois _/_ em mais duas abas do navegador, verá de fato que as requisições para _/_ não precisam esperar _/sleep_ terminar. No entanto, como mencionamos, isso eventualmente sobrecarregará o sistema porque você estaria criando novas threads sem qualquer limite.

Você também pode se lembrar do Capítulo 17 de que esta é exatamente o tipo de situação em que async e await realmente se destacam! Tenha isso em mente enquanto construímos o pool de threads e pense em como as coisas seriam diferentes ou iguais com async.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Criando um número finito de threads

Queremos que nosso pool de threads funcione de forma semelhante e familiar, de modo que mudar de threads para um pool de threads não exija grandes alterações no código que usa nossa API. A Listagem 21-12 mostra a interface hipotética para uma struct `ThreadPool` que queremos usar em vez de `thread::spawn`.

**Arquivo: src/main.rs**

```rust,ignore,does_not_compile
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

<a id="listagem-21-12"></a>

[Listagem 21-12](#listagem-21-12): Nossa interface ideal de `ThreadPool`

Usamos `ThreadPool::new` para criar um novo pool de threads com um número configurável de threads, neste caso quatro. Então, no loop `for`, `pool.execute` tem uma interface semelhante a `thread::spawn` no sentido de que recebe um closure que o pool deve executar para cada stream. Precisamos implementar `pool.execute` para que ele receba o closure e o entregue a uma thread no pool para executar. Este código ainda não compilará, mas tentaremos para que o compilador nos oriente sobre como corrigi-lo.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Construindo `ThreadPool` usando desenvolvimento orientado pelo compilador

Faça as mudanças da Listagem 21-12 em _src/main.rs_ e então usemos os erros do compilador de `cargo check` para orientar nosso desenvolvimento. Este é o primeiro erro que obtemos:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`

For more information about this error, try `rustc --explain E0433`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```

Ótimo! Este erro nos diz que precisamos de um tipo ou módulo `ThreadPool`, então vamos construir um agora. Nossa implementação de `ThreadPool` será independente do tipo de trabalho que nosso servidor web está fazendo. Então, vamos mudar o crate `hello` de um crate binário para um crate de biblioteca para abrigar nossa implementação de `ThreadPool`. Depois de mudar para um crate de biblioteca, também poderíamos usar a biblioteca de pool de threads separada para qualquer trabalho que quisermos fazer usando um pool de threads, não apenas para servir requisições web.

Crie um arquivo _src/lib.rs_ que contenha o seguinte, que é a definição mais simples de uma struct `ThreadPool` que podemos ter por enquanto:

**Arquivo: src/lib.rs**

```rust,noplayground
pub struct ThreadPool;
```

Em seguida, edite o arquivo _main.rs_ para trazer `ThreadPool` para o escopo a partir do crate de biblioteca adicionando o seguinte código no topo de _src/main.rs_:

**Arquivo: src/main.rs**

```rust,ignore
use hello::ThreadPool;
```

<a id="listagem-no-listing-01"></a>

[Listagem no-listing-01](#listagem-no-listing-01): Definindo a struct `ThreadPool` no crate de biblioteca

Este código ainda não funcionará, mas vamos verificá-lo novamente para obter o próximo erro que precisamos resolver:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```

Este erro indica que em seguida precisamos criar uma função associada chamada `new` para `ThreadPool`. Também sabemos que `new` precisa ter um parâmetro que possa aceitar `4` como argumento e deve retornar uma instância de `ThreadPool`. Vamos implementar a função `new` mais simples que terá essas características:

**Arquivo: src/lib.rs**

```rust,noplayground
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

<a id="listagem-no-listing-02"></a>

[Listagem no-listing-02](#listagem-no-listing-02): Implementando a função associada `new` para `ThreadPool`

Escolhemos `usize` como o tipo do parâmetro `size` porque sabemos que um número negativo de threads não faz sentido. Também sabemos que usaremos esse `4` como o número de elementos em uma coleção de threads, que é para o que o tipo `usize` serve, como discutido na seção [“Tipos inteiros”](#tipos-inteiros) do Capítulo 3.

Vamos verificar o código novamente:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |         -----^^^^^^^ method not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```

Agora o erro ocorre porque não temos um método `execute` em `ThreadPool`. Lembre-se da seção [“Criando um número finito de threads”](#creating-a-finite-number-of-threads) de que decidimos que nosso pool de threads deveria ter uma interface semelhante a `thread::spawn`. Além disso, implementaremos a função `execute` para que ela receba o closure que lhe é dado e o entregue a uma thread ociosa no pool para executar.

Definiremos o método `execute` em `ThreadPool` para receber um closure como parâmetro. Lembre-se da seção [“Movendo valores capturados para fora de closures”](#movendo-valores-capturados-para-fora-de-closures) do Capítulo 13 de que podemos receber closures como parâmetros com três traits diferentes: `Fn`, `FnMut` e `FnOnce`. Precisamos decidir qual tipo de closure usar aqui. Sabemos que acabaremos fazendo algo semelhante à implementação de `thread::spawn` da biblioteca padrão, então podemos olhar quais limites a assinatura de `thread::spawn` impõe em seu parâmetro. A documentação nos mostra o seguinte:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

O parâmetro de tipo `F` é o que nos interessa aqui; o parâmetro de tipo `T` está relacionado ao valor de retorno, e não nos preocupamos com ele. Podemos ver que `spawn` usa `FnOnce` como o trait bound em `F`. Provavelmente é o que queremos também, porque eventualmente passaremos o argumento que recebemos em `execute` para `spawn`. Podemos ter ainda mais confiança de que `FnOnce` é a trait que queremos usar porque a thread para executar uma requisição executará o closure dessa requisição apenas uma vez, o que corresponde ao `Once` em `FnOnce`.

O parâmetro de tipo `F` também tem o trait bound `Send` e o lifetime bound `'static`, que são úteis em nossa situação: precisamos de `Send` para transferir o closure de uma thread para outra e `'static` porque não sabemos quanto tempo a thread levará para executar. Vamos criar um método `execute` em `ThreadPool` que receberá um parâmetro genérico de tipo `F` com esses bounds:

**Arquivo: src/lib.rs**

```rust,noplayground
pub struct ThreadPool;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
```

<a id="listagem-no-listing-03"></a>

[Listagem no-listing-03](#listagem-no-listing-03): Definindo o método `execute` em `ThreadPool`

Ainda usamos `()` depois de `FnOnce` porque este `FnOnce` representa um closure que não recebe parâmetros e retorna o tipo unitário `()`. Assim como nas definições de função, o tipo de retorno pode ser omitido da assinatura, mas mesmo que não tenhamos parâmetros, ainda precisamos dos parênteses.

Novamente, esta é a implementação mais simples do método `execute`: ele não faz nada, mas estamos apenas tentando fazer nosso código compilar. Vamos verificá-lo novamente:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.24s
```

Compila! Mas observe que, se você tentar `cargo run` e fizer uma requisição no navegador, verá os erros no navegador que vimos no início do capítulo. Nossa biblioteca ainda não está realmente chamando o closure passado para `execute`!

> Nota: Você pode ouvir um ditado sobre linguagens com compiladores rigorosos, como Haskell e Rust: “Se o código compila, funciona.” Mas esse ditado não é universalmente verdadeiro. Nosso projeto compila, mas não faz absolutamente nada! Se estivéssemos construindo um projeto real e completo, este seria um bom momento para começar a escrever testes de unidade para verificar que o código compila _e_ tem o comportamento que queremos.

Considere: o que seria diferente aqui se fôssemos executar uma future em vez de um closure?

#### Validando o número de threads em `new`

Não estamos fazendo nada com os parâmetros de `new` e `execute`. Vamos implementar os corpos dessas funções com o comportamento que queremos. Para começar, vamos pensar em `new`. Anteriormente escolhemos um tipo sem sinal para o parâmetro `size` porque um pool com um número negativo de threads não faz sentido. No entanto, um pool com zero threads também não faz sentido, mas zero é um `usize` perfeitamente válido. Adicionaremos código para verificar que `size` é maior que zero antes de retornarmos uma instância de `ThreadPool`, e faremos o programa entrar em pânico se receber um zero usando a macro `assert!`, como mostrado na Listagem 21-13.

**Arquivo: src/lib.rs**

```rust,noplayground
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --snip--
```

<a id="listagem-21-13"></a>

[Listagem 21-13](#listagem-21-13): Implementando `ThreadPool::new` para entrar em pânico se `size` for zero

Também adicionamos alguma documentação para nosso `ThreadPool` com comentários de documentação. Observe que seguimos boas práticas de documentação adicionando uma seção que destaca as situações em que nossa função pode entrar em pânico, como discutido no Capítulo 14. Tente executar `cargo doc --open` e clicar na struct `ThreadPool` para ver como ficam os docs gerados para `new`!

Em vez de adicionar a macro `assert!` como fizemos aqui, poderíamos mudar `new` para `build` e retornar um `Result` como fizemos com `Config::build` no projeto de I/O na Listagem 12-9. Mas decidimos neste caso que tentar criar um pool de threads sem nenhuma thread deve ser um erro irrecuperável. Se você estiver se sentindo ambicioso, tente escrever uma função chamada `build` com a seguinte assinatura para comparar com a função `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Criando espaço para armazenar as threads

Agora que temos uma forma de saber que temos um número válido de threads para armazenar no pool, podemos criar essas threads e armazená-las na struct `ThreadPool` antes de retornar a struct. Mas como “armazenamos” uma thread? Vamos olhar novamente a assinatura de `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

A função `spawn` retorna um `JoinHandle<T>`, onde `T` é o tipo que o closure retorna. Vamos tentar usar `JoinHandle` também e ver o que acontece. No nosso caso, os closures que passamos ao pool de threads tratarão a conexão e não retornarão nada, então `T` será o tipo unitário `()`.

O código na Listagem 21-14 compilará, mas ainda não cria nenhuma thread. Mudamos a definição de `ThreadPool` para conter um vetor de instâncias de `thread::JoinHandle<()>`, inicializamos o vetor com capacidade de `size`, configuramos um loop `for` que executará algum código para criar as threads e retornamos uma instância de `ThreadPool` contendo-as.

**Arquivo: src/lib.rs**

```rust,ignore,not_desired_behavior
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool { threads }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
```

<a id="listagem-21-14"></a>

[Listagem 21-14](#listagem-21-14): Criando um vetor para `ThreadPool` armazenar as threads

Trouxemos `std::thread` para o escopo no crate de biblioteca porque estamos usando `thread::JoinHandle` como o tipo dos itens no vetor em `ThreadPool`.

Depois que um `size` válido é recebido, nosso `ThreadPool` cria um novo vetor que pode conter `size` itens. A função `with_capacity` realiza a mesma tarefa que `Vec::new`, mas com uma diferença importante: ela pré-aloca espaço no vetor. Como sabemos que precisamos armazenar `size` elementos no vetor, fazer essa alocação antecipadamente é ligeiramente mais eficiente do que usar `Vec::new`, que redimensiona a si mesmo conforme os elementos são inseridos.

Quando você executar `cargo check` novamente, deve ter sucesso.

<!-- Old headings. Do not remove or links may break. -->
<a id="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Enviando código do `ThreadPool` para uma thread

Deixamos um comentário no loop `for` da Listagem 21-14 sobre a criação de threads. Aqui, veremos como realmente criamos threads. A biblioteca padrão fornece `thread::spawn` como uma forma de criar threads, e `thread::spawn` espera receber algum código que a thread deve executar assim que a thread é criada. No entanto, no nosso caso, queremos criar as threads e fazê-las _esperar_ por código que enviaremos depois. A implementação de threads da biblioteca padrão não inclui nenhuma forma de fazer isso; temos de implementá-lo manualmente.

Implementaremos esse comportamento introduzindo uma nova estrutura de dados entre o `ThreadPool` e as threads que gerenciará esse novo comportamento. Chamaremos essa estrutura de dados de _Worker_, que é um termo comum em implementações de pooling. O `Worker` pega o código que precisa ser executado e executa o código em sua thread.

Pense nas pessoas trabalhando na cozinha de um restaurante: os trabalhadores esperam até que os pedidos cheguem dos clientes, e então são responsáveis por pegar esses pedidos e atendê-los.

Em vez de armazenar um vetor de instâncias de `JoinHandle<()>` no pool de threads, armazenaremos instâncias da struct `Worker`. Cada `Worker` armazenará uma única instância de `JoinHandle<()>`. Então, implementaremos um método em `Worker` que receberá um closure de código para executar e o enviará para a thread já em execução para execução. Também daremos a cada `Worker` um `id` para que possamos distinguir entre as diferentes instâncias de `Worker` no pool ao registrar logs ou depurar.

Este é o novo processo que acontecerá quando criarmos um `ThreadPool`. Implementaremos o código que envia o closure para a thread depois de termos o `Worker` configurado dessa forma:

1. Definir uma struct `Worker` que contenha um `id` e um `JoinHandle<()>`.
2. Mudar `ThreadPool` para conter um vetor de instâncias de `Worker`.
3. Definir uma função `Worker::new` que recebe um número `id` e retorna uma instância de `Worker` que contém o `id` e uma thread criada com um closure vazio.
4. Em `ThreadPool::new`, usar o contador do loop `for` para gerar um `id`, criar um novo `Worker` com esse `id` e armazenar o `Worker` no vetor.

Se você estiver a fim de um desafio, tente implementar essas mudanças por conta própria antes de olhar o código na Listagem 21-15.

Pronto? Aqui está a Listagem 21-15 com uma forma de fazer as modificações anteriores.

**Arquivo: src/lib.rs**

```rust,noplayground
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

<a id="listagem-21-15"></a>

[Listagem 21-15](#listagem-21-15): Modificando `ThreadPool` para conter instâncias de `Worker` em vez de conter threads diretamente

Mudamos o nome do campo em `ThreadPool` de `threads` para `workers` porque agora ele contém instâncias de `Worker` em vez de instâncias de `JoinHandle<()>`. Usamos o contador no loop `for` como argumento para `Worker::new`, e armazenamos cada novo `Worker` no vetor chamado `workers`.

O código externo (como nosso servidor em _src/main.rs_) não precisa saber os detalhes de implementação sobre o uso de uma struct `Worker` dentro de `ThreadPool`, então tornamos a struct `Worker` e sua função `new` privadas. A função `Worker::new` usa o `id` que lhe damos e armazena uma instância de `JoinHandle<()>` que é criada ao criar uma nova thread usando um closure vazio.

> Nota: Se o sistema operacional não puder criar uma thread porque não há recursos suficientes do sistema, `thread::spawn` entrará em pânico. Isso fará com que todo o nosso servidor entre em pânico, mesmo que a criação de algumas threads possa ter sucesso. Por simplicidade, esse comportamento está bem, mas em uma implementação de pool de threads de produção, você provavelmente quereria usar [`std::thread::Builder`](https://doc.rust-lang.org/std/thread/struct.Builder.html) e seu método [`spawn`](https://doc.rust-lang.org/std/thread/struct.Builder.html#method.spawn), que retorna `Result`, em vez disso.

Este código compilará e armazenará o número de instâncias de `Worker` que especificamos como argumento para `ThreadPool::new`. Mas ainda _não_ estamos processando o closure que recebemos em `execute`. Vamos ver como fazer isso em seguida.

#### Enviando requisições para threads via channels

O próximo problema que enfrentaremos é que os closures passados para `thread::spawn` não fazem absolutamente nada. Atualmente, recebemos o closure que queremos executar no método `execute`. Mas precisamos dar a `thread::spawn` um closure para executar quando criamos cada `Worker` durante a criação do `ThreadPool`.

Queremos que as structs `Worker` que acabamos de criar busquem o código a executar em uma fila mantida no `ThreadPool` e enviem esse código para sua thread para executar.

Os channels que aprendemos no Capítulo 16 — uma forma simples de comunicar entre duas threads — seriam perfeitos para este caso de uso. Usaremos um channel para funcionar como a fila de jobs, e `execute` enviará um job do `ThreadPool` para as instâncias de `Worker`, que enviarão o job para sua thread. Este é o plano:

1. O `ThreadPool` criará um channel e manterá o sender.
2. Cada `Worker` manterá o receiver.
3. Criaremos uma nova struct `Job` que conterá os closures que queremos enviar pelo channel.
4. O método `execute` enviará o job que quer executar pelo sender.
5. Em sua thread, o `Worker` fará loop sobre seu receiver e executará os closures de quaisquer jobs que receber.

Vamos começar criando um channel em `ThreadPool::new` e mantendo o sender na instância de `ThreadPool`, como mostrado na Listagem 21-16. A struct `Job` não contém nada por enquanto, mas será o tipo de item que estamos enviando pelo channel.

**Arquivo: src/lib.rs**

```rust,noplayground
use std::{sync::mpsc, thread};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

<a id="listagem-21-16"></a>

[Listagem 21-16](#listagem-21-16): Modificando `ThreadPool` para armazenar o sender de um channel que transmite instâncias de `Job`

Em `ThreadPool::new`, criamos nosso novo channel e fazemos o pool manter o sender. Isso compilará com sucesso.

Vamos tentar passar um receiver do channel para cada `Worker` conforme o pool de threads cria o channel. Sabemos que queremos usar o receiver na thread que as instâncias de `Worker` criam, então referenciaremos o parâmetro `receiver` no closure. O código na Listagem 21-17 ainda não compilará completamente.

**Arquivo: src/lib.rs**

```rust,ignore,does_not_compile
impl ThreadPool {
    // --snip--
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}

// --snip--

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

<a id="listagem-21-17"></a>

[Listagem 21-17](#listagem-21-17): Passando o receiver para cada `Worker`

Fizemos algumas mudanças pequenas e diretas: passamos o receiver para `Worker::new` e então o usamos dentro do closure.

Quando tentamos verificar este código, obtemos este erro:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
25 |         for id in 0..size {
   |         ----------------- inside of this loop
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop
   |
note: consider changing this parameter type in method `new` to borrow instead if owning the value isn't necessary
  --> src/lib.rs:47:33
   |
47 |     fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
   |        --- in this method       ^^^^^^^^^^^^^^^^^^^ this parameter takes ownership of the value
help: consider moving the expression out of the loop so it is only moved once
   |
25 ~         let mut value = Worker::new(id, receiver);
26 ~         for id in 0..size {
27 ~             workers.push(value);
   |

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` (lib) due to 1 previous error
```

O código está tentando passar `receiver` para várias instâncias de `Worker`. Isso não funcionará, como você se lembrará do Capítulo 16: a implementação de channel que o Rust fornece é vários _produtores_, um _consumidor_. Isso significa que não podemos simplesmente clonar a extremidade consumidora do channel para corrigir este código. Também não queremos enviar uma mensagem várias vezes para vários consumidores; queremos uma lista de mensagens com várias instâncias de `Worker` de modo que cada mensagem seja processada uma vez.

Além disso, retirar um job da fila do channel envolve mutar o `receiver`, então as threads precisam de uma forma segura de compartilhar e modificar `receiver`; caso contrário, podemos obter condições de corrida (como coberto no Capítulo 16).

Lembre-se dos smart pointers thread-safe discutidos no Capítulo 16: para compartilhar propriedade entre várias threads e permitir que as threads mutem o valor, precisamos usar `Arc<Mutex<T>>`. O tipo `Arc` permitirá que várias instâncias de `Worker` possuam o receiver, e `Mutex` garantirá que apenas um `Worker` obtenha um job do receiver por vez. A Listagem 21-18 mostra as mudanças que precisamos fazer.

**Arquivo: src/lib.rs**

```rust,noplayground
use std::{
    sync::{Arc, Mutex, mpsc},
    thread,
};
// --snip--

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}

// --snip--

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

<a id="listagem-21-18"></a>

[Listagem 21-18](#listagem-21-18): Compartilhando o receiver entre as instâncias de `Worker` usando `Arc` e `Mutex`

Em `ThreadPool::new`, colocamos o receiver em um `Arc` e um `Mutex`. Para cada novo `Worker`, clonamos o `Arc` para aumentar a contagem de referências para que as instâncias de `Worker` possam compartilhar a propriedade do receiver.

Com essas mudanças, o código compila! Estamos chegando lá!

#### Implementando o método `execute`

Vamos finalmente implementar o método `execute` em `ThreadPool`. Também mudaremos `Job` de uma struct para um alias de tipo para um trait object que contém o tipo de closure que `execute` recebe. Como discutido na seção [“Sinônimos de tipo e aliases de tipo”](#sinônimos-de-tipo-e-aliases-de-tipo) do Capítulo 20, aliases de tipo nos permitem tornar tipos longos mais curtos para facilitar o uso. Veja a Listagem 21-19.

**Arquivo: src/lib.rs**

```rust,noplayground
// --snip--

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}
```

<a id="listagem-21-19"></a>

[Listagem 21-19](#listagem-21-19): Criando um alias de tipo `Job` para um `Box` que contém cada closure e então enviando o job pelo channel

Depois de criar uma nova instância de `Job` usando o closure que recebemos em `execute`, enviamos esse job pela extremidade de envio do channel. Estamos chamando `unwrap` em `send` para o caso de o envio falhar. Isso pode acontecer se, por exemplo, pararmos todas as nossas threads de executar, o que significa que a extremidade de recebimento parou de receber novas mensagens. No momento, não podemos parar nossas threads de executar: nossas threads continuam executando enquanto o pool existir. O motivo de usarmos `unwrap` é que sabemos que o caso de falha não acontecerá, mas o compilador não sabe disso.

Mas ainda não terminamos! No `Worker`, nosso closure passado para `thread::spawn` ainda apenas _referencia_ a extremidade de recebimento do channel. Em vez disso, precisamos que o closure faça loop para sempre, pedindo à extremidade de recebimento do channel um job e executando o job quando obtiver um. Vamos fazer a mudança mostrada na Listagem 21-20 em `Worker::new`.

**Arquivo: src/lib.rs**

```rust,noplayground
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let job = receiver.lock().unwrap().recv().unwrap();

                println!("Worker {id} got a job; executing.");

                job();
            }
        });

        Worker { id, thread }
    }
}
```

<a id="listagem-21-20"></a>

[Listagem 21-20](#listagem-21-20): Recebendo e executando os jobs na thread da instância de `Worker`

Aqui, primeiro chamamos `lock` no `receiver` para adquirir o mutex e então chamamos `unwrap` para entrar em pânico em quaisquer erros. Adquirir um lock pode falhar se o mutex estiver em um estado _envenenado_, o que pode acontecer se alguma outra thread entrar em pânico enquanto mantém o lock em vez de liberar o lock. Nessa situação, chamar `unwrap` para fazer esta thread entrar em pânico é a ação correta a tomar. Sinta-se à vontade para mudar este `unwrap` para um `expect` com uma mensagem de erro que faça sentido para você.

Se obtivermos o lock no mutex, chamamos `recv` para receber um `Job` do channel. Um `unwrap` final passa além de quaisquer erros aqui também, o que pode ocorrer se a thread que mantém o sender tiver sido encerrada, de forma semelhante a como o método `send` retorna `Err` se o receiver for encerrado.

A chamada a `recv` bloqueia, então se ainda não houver job, a thread atual esperará até que um job fique disponível. O `Mutex<T>` garante que apenas uma thread `Worker` por vez esteja tentando solicitar um job.

Nosso pool de threads agora está em um estado funcional! Execute `cargo run` e faça algumas requisições:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Sucesso! Agora temos um pool de threads que executa conexões de forma assíncrona. Nunca são criadas mais de quatro threads, então nosso sistema não ficará sobrecarregado se o servidor receber muitas requisições. Se fizermos uma requisição para _/sleep_, o servidor poderá atender outras requisições fazendo outra thread executá-las.

> Nota: Se você abrir _/sleep_ em várias janelas do navegador simultaneamente, elas podem carregar uma de cada vez em intervalos de cinco segundos. Alguns navegadores executam várias instâncias da mesma requisição em sequência por motivos de cache. Essa limitação não é causada pelo nosso servidor web.

Este é um bom momento para pausar e considerar como o código nas Listagens 21-18, 21-19 e 21-20 seria diferente se estivéssemos usando futures em vez de um closure para o trabalho a ser feito. Quais tipos mudariam? Como seriam as assinaturas dos métodos, se mudassem? Quais partes do código permaneceriam iguais?

Depois de aprender sobre o loop `while let` nos Capítulos 17 e 19, você pode estar se perguntando por que não escrevemos o código da thread `Worker` como mostrado na Listagem 21-21.

**Arquivo: src/lib.rs**

```rust,ignore,not_desired_behavior
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!("Worker {id} got a job; executing.");

                job();
            }
        });

        Worker { id, thread }
    }
}
```

<a id="listagem-21-21"></a>

[Listagem 21-21](#listagem-21-21): Uma implementação alternativa de `Worker::new` usando `while let`

Este código compila e executa, mas não resulta no comportamento de threading desejado: uma requisição lenta ainda fará com que outras requisições esperem para serem processadas. O motivo é um tanto sutil: a struct `Mutex` não tem um método público `unlock` porque a propriedade do lock é baseada no lifetime do `MutexGuard<T>` dentro do `LockResult<MutexGuard<T>>` que o método `lock` retorna. Em tempo de compilação, o borrow checker pode então impor a regra de que um recurso protegido por um `Mutex` não pode ser acessado a menos que mantenhamos o lock. No entanto, essa implementação também pode resultar no lock sendo mantido por mais tempo do que o pretendido se não prestarmos atenção ao lifetime do `MutexGuard<T>`.

O código na Listagem 21-20 que usa `let job = receiver.lock().unwrap().recv().unwrap();` funciona porque com `let`, quaisquer valores temporários usados na expressão no lado direito do sinal de igual são imediatamente descartados quando a instrução `let` termina. No entanto, `while let` (e `if let` e `match`) não descarta valores temporários até o fim do bloco associado. Na Listagem 21-21, o lock permanece mantido durante a duração da chamada a `job()`, o que significa que outras instâncias de `Worker` não podem receber jobs.
