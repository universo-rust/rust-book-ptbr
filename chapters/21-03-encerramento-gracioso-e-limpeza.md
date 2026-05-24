---
title: "Encerramento gracioso e limpeza"
chapter_code: 21-03
slug: encerramento-gracioso-e-limpeza
---

# Encerramento Gracioso e Limpeza

O código da Listagem 21-20 está respondendo a requisições de forma assíncrona por meio do uso de um pool de threads, como pretendíamos. Recebemos alguns avisos sobre os campos `workers`, `id` e `thread` que não estamos usando de forma direta, o que nos lembra que não estamos limpando nada. Quando usamos o método menos elegante <kbd>Ctrl</kbd>+<kbd>C</kbd> para interromper a thread principal, todas as outras threads também são paradas imediatamente, mesmo que estejam no meio de atender uma requisição.

A seguir, implementaremos a trait `Drop` para chamar `join` em cada uma das threads no pool para que possam terminar as requisições em que estão trabalhando antes de fechar. Em seguida, implementaremos uma forma de dizer às threads que devem parar de aceitar novas requisições e encerrar. Para ver este código em ação, modificaremos nosso servidor para aceitar apenas duas requisições antes de encerrar graciosamente seu pool de threads.

Uma coisa a observar enquanto avançamos: nada disso afeta as partes do código que lidam com a execução dos closures, então tudo aqui seria o mesmo se estivéssemos usando um pool de threads para um runtime async.

## Implementando a Trait `Drop` em `ThreadPool`

Vamos começar implementando `Drop` em nosso pool de threads. Quando o pool for descartado, nossas threads devem todas fazer join para garantir que terminem seu trabalho. A Listagem 21-22 mostra uma primeira tentativa de implementação de `Drop`; este código ainda não funcionará completamente.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

<a id="listagem-21-22"></a>

[Listagem 21-22](#listagem-21-22): Fazendo join em cada thread quando o pool de threads sai de escopo

Primeiro, fazemos loop por cada um dos `workers` do pool de threads. Usamos `&mut` para isso porque `self` é uma referência mutável e também precisamos poder mutar `worker`. Para cada `worker`, imprimimos uma mensagem dizendo que esta instância particular de `Worker` está encerrando e, em seguida, chamamos `join` na thread da instância `Worker`. Se a chamada a `join` falhar, usamos `unwrap` para fazer o Rust entrar em pânico e ir para um encerramento não gracioso.

Aqui está o erro que obtemos ao compilar este código:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ ------ `worker.thread` moved due to this method call
   |             |
   |             move occurs because `worker.thread` has type `JoinHandle<()>`, which does not implement the `Copy` trait
   |
note: `JoinHandle::<T>::join` takes ownership of the receiver `self`, which moves `worker.thread`
  --> /rustc/1159e78c4747b02ef996e55082b704c09b970588/library/std/src/thread/mod.rs:1921:17

For more information about this error, try `rustc --explain E0507`.
error: could not compile `hello` (lib) due to 1 previous error
```

O erro nos diz que não podemos chamar `join` porque só temos um empréstimo mutável de cada `worker` e `join` toma ownership de seu argumento. Para resolver este problema, precisamos mover a thread para fora da instância `Worker` que possui `thread` para que `join` possa consumir a thread. Uma forma de fazer isso é tomar a mesma abordagem que tomamos na Listagem 18-15. Se `Worker` mantivesse um `Option<thread::JoinHandle<()>>`, poderíamos chamar o método `take` no `Option` para mover o valor para fora da variante `Some` e deixar uma variante `None` em seu lugar. Em outras palavras, um `Worker` em execução teria uma variante `Some` em `thread`, e quando quiséssemos limpar um `Worker`, substituiríamos `Some` por `None` para que o `Worker` não tivesse uma thread para executar.

No entanto, a *única* vez em que isso surgiria seria ao descartar o `Worker`. Em troca, teríamos que lidar com um `Option<thread::JoinHandle<()>>` em qualquer lugar em que acessássemos `worker.thread`. O Rust idiomático usa `Option` bastante, mas quando você se encontra envolvendo algo que sabe que estará sempre presente em um `Option` como contorno como este, é uma boa ideia procurar abordagens alternativas para tornar seu código mais limpo e menos propenso a erros.

Neste caso, existe uma alternativa melhor: o método `Vec::drain`. Ele aceita um parâmetro de intervalo para especificar quais itens remover do vetor e retorna um iterador desses itens. Passar a sintaxe de intervalo `..` removerá *todos* os valores do vetor.

Portanto, precisamos atualizar a implementação de `drop` de `ThreadPool` assim:

**Arquivo: src/lib.rs**

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in self.workers.drain(..) {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

Isso resolve o erro de compilador e não requer nenhuma outra alteração em nosso código. Observe que, como `drop` pode ser chamado ao entrar em pânico, o `unwrap` também pode entrar em pânico e causar um pânico duplo, o que imediatamente trava o programa e encerra qualquer limpeza em andamento. Isso é aceitável para um programa de exemplo, mas não é recomendado para código de produção.

## Sinalizando às Threads para Parar de Escutar por Trabalhos

Com todas as alterações que fizemos, nosso código compila sem nenhum aviso. No entanto, a má notícia é que este código ainda não funciona da forma que queremos. A chave é a lógica nos closures executados pelas threads das instâncias `Worker`: no momento, chamamos `join`, mas isso não encerrará as threads, porque elas fazem `loop` para sempre procurando por trabalhos. Se tentarmos descartar nosso `ThreadPool` com nossa implementação atual de `drop`, a thread principal ficará bloqueada para sempre, esperando a primeira thread terminar.

Para corrigir este problema, precisaremos de uma alteração na implementação de `drop` de `ThreadPool` e depois uma alteração no loop de `Worker`.

Primeiro, alteraremos a implementação de `drop` de `ThreadPool` para descartar explicitamente o `sender` antes de esperar as threads terminarem. A Listagem 21-23 mostra as alterações em `ThreadPool` para descartar explicitamente `sender`. Diferentemente da thread, aqui *precisamos* usar um `Option` para poder mover `sender` para fora de `ThreadPool` com `Option::take`.

**Arquivo: src/lib.rs**

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --snip--

impl ThreadPool {
    // --snip--

    pub fn new(size: usize) -> ThreadPool {
        // --snip--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in self.workers.drain(..) {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

<a id="listagem-21-23"></a>

[Listagem 21-23](#listagem-21-23): Descartando explicitamente `sender` antes de fazer join nas threads `Worker`

Descartar `sender` fecha o canal, o que indica que não serão enviadas mais mensagens. Quando isso acontece, todas as chamadas a `recv` que as instâncias `Worker` fazem no loop infinito retornarão um erro. Na Listagem 21-24, alteramos o loop de `Worker` para sair graciosamente do loop nesse caso, o que significa que as threads terminarão quando a implementação de `drop` de `ThreadPool` chamar `join` nelas.

**Arquivo: src/lib.rs**

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let message = receiver.lock().unwrap().recv();

                match message {
                    Ok(job) => {
                        println!("Worker {id} got a job; executing.");

                        job();
                    }
                    Err(_) => {
                        println!("Worker {id} disconnected; shutting down.");
                        break;
                    }
                }
            }
        });

        Worker { id, thread }
    }
}
```

<a id="listagem-21-24"></a>

[Listagem 21-24](#listagem-21-24): Saindo explicitamente do loop quando `recv` retorna um erro

Para ver este código em ação, vamos modificar `main` para aceitar apenas duas requisições antes de encerrar graciosamente o servidor, conforme mostrado na Listagem 21-25.

**Arquivo: src/main.rs**

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}
```

<a id="listagem-21-25"></a>

[Listagem 21-25](#listagem-21-25): Encerrando o servidor depois de atender duas requisições saindo do loop

Você não quereria que um servidor web do mundo real encerrasse depois de atender apenas duas requisições. Este código apenas demonstra que o encerramento gracioso e a limpeza estão funcionando.

O método `take` é definido na trait `Iterator` e limita a iteração aos primeiros dois itens no máximo. O `ThreadPool` sairá de escopo no final de `main`, e a implementação de `drop` será executada.

Inicie o servidor com `cargo run` e faça três requisições. A terceira requisição deve dar erro e, em seu terminal, você deverá ver uma saída semelhante a esta:

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Você pode ver uma ordenação diferente de IDs de `Worker` e mensagens impressas. Podemos ver como este código funciona pelas mensagens: as instâncias `Worker` 0 e 3 receberam as duas primeiras requisições. O servidor parou de aceitar conexões depois da segunda conexão, e a implementação de `Drop` em `ThreadPool` começa a executar antes mesmo de `Worker 3` iniciar seu trabalho. Descartar o `sender` desconecta todas as instâncias `Worker` e diz a elas para encerrar. As instâncias `Worker` imprimem uma mensagem quando se desconectam e, em seguida, o pool de threads chama `join` para esperar cada thread `Worker` terminar.

Observe um aspecto interessante desta execução particular: o `ThreadPool` descartou o `sender` e, antes de qualquer `Worker` receber um erro, tentamos fazer join em `Worker 0`. `Worker 0` ainda não tinha recebido um erro de `recv`, então a thread principal ficou bloqueada, esperando `Worker 0` terminar. Enquanto isso, `Worker 3` recebeu um trabalho e então todas as threads receberam um erro. Quando `Worker 0` terminou, a thread principal esperou o restante das instâncias `Worker` terminar. Nesse ponto, todas já tinham saído de seus loops e parado.

Parabéns! Agora concluímos nosso projeto; temos um servidor web básico que usa um pool de threads para responder de forma assíncrona. Somos capazes de realizar um encerramento gracioso do servidor, que limpa todas as threads no pool.

Aqui está o código completo para referência:

**Arquivo: src/main.rs**

```rust
use hello::ThreadPool;
use std::{
    fs,
    io::{BufReader, prelude::*},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
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

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

**Arquivo: src/lib.rs**

```rust
use std::{
    sync::{Arc, Mutex, mpsc},
    thread,
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

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

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let message = receiver.lock().unwrap().recv();

                match message {
                    Ok(job) => {
                        println!("Worker {id} got a job; executing.");

                        job();
                    }
                    Err(_) => {
                        println!("Worker {id} disconnected; shutting down.");
                        break;
                    }
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

Poderíamos fazer mais aqui! Se quiser continuar aprimorando este projeto, aqui estão algumas ideias:

- Adicionar mais documentação a `ThreadPool` e seus métodos públicos.
- Adicionar testes da funcionalidade da biblioteca.
- Alterar chamadas a `unwrap` para tratamento de erros mais robusto.
- Usar `ThreadPool` para executar alguma tarefa que não seja servir requisições web.
- Encontrar um crate de pool de threads em [crates.io](https://crates.io/) e implementar um servidor web semelhante usando o crate em vez disso. Em seguida, compare sua API e robustez com o pool de threads que implementamos.

## Resumo

Muito bem! Você chegou ao fim do livro! Queremos agradecer por nos acompanhar neste tour por Rust. Agora você está pronto para implementar seus próprios projetos Rust e ajudar com projetos de outras pessoas. Lembre-se de que há uma comunidade acolhedora de outros Rustaceans que adorariam ajudá-lo com quaisquer desafios que encontrar em sua jornada com Rust.
