---
title: "Servidor monothread"
chapter_code: 21-01
slug: servidor-monothread
---

# Construindo um Servidor Web Monothread

Começaremos fazendo um servidor web monothread funcionar. Antes de começarmos, vamos dar uma visão geral rápida dos protocolos envolvidos na construção de servidores web. Os detalhes desses protocolos estão além do escopo deste livro, mas uma breve visão geral lhe dará as informações de que precisa.

Os dois principais protocolos envolvidos em servidores web são o _Hypertext Transfer Protocol_ _(HTTP)_ e o _Transmission Control Protocol_ _(TCP)_. Ambos os protocolos são protocolos de _requisição-resposta_, o que significa que um _cliente_ inicia requisições e um _servidor_ escuta as requisições e fornece uma resposta ao cliente. O conteúdo dessas requisições e respostas é definido pelos protocolos.

TCP é o protocolo de nível mais baixo que descreve os detalhes de como a informação vai de um servidor para outro, mas não especifica qual é essa informação. HTTP se constrói sobre TCP definindo o conteúdo das requisições e respostas. Tecnicamente é possível usar HTTP com outros protocolos, mas na grande maioria dos casos, HTTP envia seus dados sobre TCP. Trabalharemos com os bytes brutos de requisições e respostas TCP e HTTP.

## Escutando a Conexão TCP

Nosso servidor web precisa escutar uma conexão TCP, então essa é a primeira parte com a qual trabalharemos. A biblioteca padrão oferece um módulo `std::net` que nos permite fazer isso. Vamos criar um novo projeto da forma usual:

```bash
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Agora insira o código da Listagem 21-1 em _src/main.rs_ para começar. Este código escutará no endereço local `127.0.0.1:7878` por streams TCP de entrada. Quando receber um stream de entrada, imprimirá `Connection established!`.

**Arquivo: src/main.rs**

```rust
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}
```

<a id="listagem-21-1"></a>

[Listagem 21-1](#listagem-21-1): Escutando streams de entrada e imprimindo uma mensagem quando recebemos um stream

Usando `TcpListener`, podemos escutar conexões TCP no endereço `127.0.0.1:7878`. No endereço, a seção antes dos dois pontos é um endereço IP que representa seu computador (isso é o mesmo em todo computador e não representa especificamente o computador dos autores), e `7878` é a porta. Escolhemos esta porta por dois motivos: HTTP normalmente não é aceito nesta porta, então nosso servidor provavelmente não entrará em conflito com nenhum outro servidor web que você possa ter rodando em sua máquina, e 7878 é _rust_ digitado em um telefone.

A função `bind` neste cenário funciona como a função `new` no sentido de que retornará uma nova instância de `TcpListener`. A função se chama `bind` porque, em redes, conectar-se a uma porta para escutar é conhecido como “fazer bind em uma porta”.

A função `bind` retorna um `Result<T, E>`, o que indica que é possível que o bind falhe, por exemplo, se executarmos duas instâncias do nosso programa e, portanto, tivermos dois programas escutando na mesma porta. Como estamos escrevendo um servidor básico apenas para fins de aprendizado, não nos preocuparemos em tratar esses tipos de erros; em vez disso, usamos `unwrap` para parar o programa se ocorrerem erros.

O método `incoming` em `TcpListener` retorna um iterador que nos dá uma sequência de streams (mais especificamente, streams do tipo `TcpStream`). Um único _stream_ representa uma conexão aberta entre o cliente e o servidor. _Conexão_ é o nome do processo completo de requisição e resposta no qual um cliente se conecta ao servidor, o servidor gera uma resposta e o servidor fecha a conexão. Assim, leremos do `TcpStream` para ver o que o cliente enviou e, em seguida, escreveremos nossa resposta no stream para enviar dados de volta ao cliente. No geral, este loop `for` processará cada conexão por turno e produzirá uma série de streams para tratarmos.

Por enquanto, nosso tratamento do stream consiste em chamar `unwrap` para encerrar nosso programa se o stream tiver algum erro; se não houver erros, o programa imprime uma mensagem. Adicionaremos mais funcionalidade para o caso de sucesso na próxima listagem. A razão pela qual podemos receber erros do método `incoming` quando um cliente se conecta ao servidor é que na verdade não estamos iterando sobre conexões. Em vez disso, estamos iterando sobre _tentativas de conexão_. A conexão pode não ser bem-sucedida por vários motivos, muitos deles específicos do sistema operacional. Por exemplo, muitos sistemas operacionais têm um limite para o número de conexões abertas simultâneas que podem suportar; novas tentativas de conexão além desse número produzirão um erro até que algumas das conexões abertas sejam fechadas.

Vamos tentar executar este código! Invoque `cargo run` no terminal e, em seguida, carregue _127.0.0.1:7878_ em um navegador web. O navegador deve mostrar uma mensagem de erro como “Connection reset” porque o servidor atualmente não está enviando nenhum dado de volta. Mas quando você olhar seu terminal, deverá ver várias mensagens que foram impressas quando o navegador se conectou ao servidor!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Às vezes você verá várias mensagens impressas para uma única requisição do navegador; a razão pode ser que o navegador está fazendo uma requisição para a página, bem como uma requisição para outros recursos, como o ícone _favicon.ico_ que aparece na aba do navegador.

Também pode ser que o navegador esteja tentando se conectar ao servidor várias vezes porque o servidor não está respondendo com nenhum dado. Quando `stream` sai de escopo e é descartado no final do loop, a conexão é fechada como parte da implementação de `drop`. Navegadores às vezes lidam com conexões fechadas tentando novamente, porque o problema pode ser temporário.

Navegadores também às vezes abrem várias conexões com o servidor sem enviar nenhuma requisição, para que, se *depois* enviarem requisições, essas requisições possam acontecer mais rapidamente. Quando isso ocorre, nosso servidor verá cada conexão, independentemente de haver requisições sobre essa conexão. Muitas versões de navegadores baseados em Chrome fazem isso, por exemplo; você pode desabilitar essa otimização usando o modo de navegação privada ou um navegador diferente.

O fator importante é que obtivemos com sucesso um handle para uma conexão TCP!

Lembre-se de parar o programa pressionando <kbd>Ctrl</kbd>+<kbd>C</kbd> quando terminar de executar uma versão particular do código. Em seguida, reinicie o programa invocando o comando `cargo run` depois de fazer cada conjunto de alterações no código para ter certeza de que está executando o código mais recente.

## Lendo a Requisição

Vamos implementar a funcionalidade para ler a requisição do navegador! Para separar as preocupações de primeiro obter uma conexão e depois tomar alguma ação com a conexão, iniciaremos uma nova função para processar conexões. Nesta nova função `handle_connection`, leremos dados do stream TCP e os imprimiremos para que possamos ver os dados sendo enviados pelo navegador. Altere o código para ficar como na Listagem 21-2.

**Arquivo: src/main.rs**

```rust
use std::{
    io::{BufReader, prelude::*},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {http_request:#?}");
}
```

<a id="listagem-21-2"></a>

[Listagem 21-2](#listagem-21-2): Lendo do `TcpStream` e imprimindo os dados

Trazemos `std::io::BufReader` e `std::io::prelude` para o escopo para obter acesso a traits e tipos que nos permitem ler e escrever no stream. No loop `for` na função `main`, em vez de imprimir uma mensagem dizendo que fizemos uma conexão, agora chamamos a nova função `handle_connection` e passamos o `stream` para ela.

Na função `handle_connection`, criamos uma nova instância de `BufReader` que envolve uma referência ao `stream`. O `BufReader` adiciona buffering gerenciando chamadas aos métodos da trait `std::io::Read` para nós.

Criamos uma variável chamada `http_request` para coletar as linhas da requisição que o navegador envia ao nosso servidor. Indicamos que queremos coletar essas linhas em um vetor adicionando a anotação de tipo `Vec<_>`.

`BufReader` implementa a trait `std::io::BufRead`, que fornece o método `lines`. O método `lines` retorna um iterador de `Result<String, std::io::Error>` dividindo o stream de dados sempre que vê um byte de nova linha. Para obter cada `String`, fazemos `map` e `unwrap` em cada `Result`. O `Result` pode ser um erro se os dados não forem UTF-8 válido ou se houve um problema ao ler do stream. Novamente, um programa de produção deveria tratar esses erros de forma mais elegante, mas estamos escolhendo parar o programa no caso de erro por simplicidade.

O navegador sinaliza o fim de uma requisição HTTP enviando dois caracteres de nova linha seguidos, então, para obter uma requisição do stream, pegamos linhas até obtermos uma linha que seja a string vazia. Depois de coletar as linhas no vetor, as imprimimos usando formatação pretty debug para que possamos dar uma olhada nas instruções que o navegador web está enviando ao nosso servidor.

Vamos tentar este código! Inicie o programa e faça uma requisição em um navegador web novamente. Observe que ainda receberemos uma página de erro no navegador, mas a saída do nosso programa no terminal agora se parecerá com isto:

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Dependendo do seu navegador, você pode obter uma saída ligeiramente diferente. Agora que estamos imprimindo os dados da requisição, podemos ver por que obtemos várias conexões de uma única requisição do navegador olhando o caminho após `GET` na primeira linha da requisição. Se as conexões repetidas estiverem todas requisitando _/_, sabemos que o navegador está tentando buscar _/_ repetidamente porque não está recebendo uma resposta do nosso programa.

Vamos analisar esses dados de requisição para entender o que o navegador está pedindo ao nosso programa.

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

## Examinando Mais de Perto uma Requisição HTTP

HTTP é um protocolo baseado em texto, e uma requisição tem este formato:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

A primeira linha é a _linha de requisição_ que contém informações sobre o que o cliente está requisitando. A primeira parte da linha de requisição indica o método sendo usado, como `GET` ou `POST`, que descreve como o cliente está fazendo esta requisição. Nosso cliente usou uma requisição `GET`, o que significa que está pedindo informação.

A próxima parte da linha de requisição é _/_, que indica o _uniform resource identifier_ _(URI)_ que o cliente está requisitando: um URI é quase, mas não exatamente, o mesmo que um _uniform resource locator_ _(URL)_. A diferença entre URIs e URLs não é importante para nossos propósitos neste capítulo, mas a especificação HTTP usa o termo _URI_, então podemos simplesmente substituir mentalmente _URL_ por _URI_ aqui.

A última parte é a versão HTTP que o cliente usa, e então a linha de requisição termina em uma sequência CRLF. (_CRLF_ significa _carriage return_ e _line feed_, que são termos da era das máquinas de escrever!) A sequência CRLF também pode ser escrita como `\r\n`, onde `\r` é um carriage return e `\n` é um line feed. A _sequência CRLF_ separa a linha de requisição do restante dos dados da requisição. Observe que, quando o CRLF é impresso, vemos uma nova linha começar em vez de `\r\n`.

Olhando os dados da linha de requisição que recebemos ao executar nosso programa até agora, vemos que `GET` é o método, _/_ é o URI da requisição e `HTTP/1.1` é a versão.

Depois da linha de requisição, as linhas restantes a partir de `Host:` em diante são cabeçalhos. Requisições `GET` não têm corpo.

Tente fazer uma requisição de um navegador diferente ou pedir um endereço diferente, como _127.0.0.1:7878/test_, para ver como os dados da requisição mudam.

Agora que sabemos o que o navegador está pedindo, vamos enviar alguns dados de volta!

## Escrevendo uma Resposta

Vamos implementar o envio de dados em resposta a uma requisição do cliente. Respostas têm o seguinte formato:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

A primeira linha é uma _linha de status_ que contém a versão HTTP usada na resposta, um código de status numérico que resume o resultado da requisição e uma frase de razão que fornece uma descrição textual do código de status. Depois da sequência CRLF vêm quaisquer cabeçalhos, outra sequência CRLF e o corpo da resposta.

Aqui está um exemplo de resposta que usa a versão HTTP 1.1 e tem um código de status 200, uma frase de razão OK, sem cabeçalhos e sem corpo:

```text
HTTP/1.1 200 OK\r\n\r\n
```

O código de status 200 é a resposta de sucesso padrão. O texto é uma resposta HTTP de sucesso minúscula. Vamos escrever isto no stream como nossa resposta a uma requisição bem-sucedida! Na função `handle_connection`, remova o `println!` que estava imprimindo os dados da requisição e substitua-o pelo código da Listagem 21-3.

**Arquivo: src/main.rs**

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let response = "HTTP/1.1 200 OK\r\n\r\n";

    stream.write_all(response.as_bytes()).unwrap();
}
```

<a id="listagem-21-3"></a>

[Listagem 21-3](#listagem-21-3): Escrevendo uma resposta HTTP de sucesso minúscula no stream

A primeira nova linha define a variável `response` que contém os dados da mensagem de sucesso. Em seguida, chamamos `as_bytes` em nossa `response` para converter os dados da string em bytes. O método `write_all` em `stream` recebe um `&[u8]` e envia esses bytes diretamente pela conexão. Como a operação `write_all` pode falhar, usamos `unwrap` em qualquer resultado de erro como antes. Novamente, em uma aplicação real, você adicionaria tratamento de erros aqui.

Com essas alterações, vamos executar nosso código e fazer uma requisição. Não estamos mais imprimindo nenhum dado no terminal, então não veremos nenhuma saída além da saída do Cargo. Quando você carregar _127.0.0.1:7878_ em um navegador web, deverá obter uma página em branco em vez de um erro. Você acabou de codificar manualmente o recebimento de uma requisição HTTP e o envio de uma resposta!

## Retornando HTML Real

Vamos implementar a funcionalidade para retornar mais do que uma página em branco. Crie o novo arquivo _hello.html_ na raiz do diretório do seu projeto, não no diretório _src_. Você pode inserir qualquer HTML que quiser; a Listagem 21-4 mostra uma possibilidade.

**Arquivo: hello.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

<a id="listagem-21-4"></a>

[Listagem 21-4](#listagem-21-4): Um arquivo HTML de exemplo para retornar em uma resposta

Este é um documento HTML5 mínimo com um cabeçalho e algum texto. Para retornar isto do servidor quando uma requisição for recebida, modificaremos `handle_connection` conforme mostrado na Listagem 21-5 para ler o arquivo HTML, adicioná-lo à resposta como corpo e enviá-lo.

**Arquivo: src/main.rs**

```rust
use std::{
    fs,
    io::{BufReader, prelude::*},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

<a id="listagem-21-5"></a>

[Listagem 21-5](#listagem-21-5): Enviando o conteúdo de *hello.html* como corpo da resposta

Adicionamos `fs` à instrução `use` para trazer o módulo de sistema de arquivos da biblioteca padrão para o escopo. O código para ler o conteúdo de um arquivo em uma string deve parecer familiar; usamos isso quando lemos o conteúdo de um arquivo para nosso projeto de E/S na Listagem 12-4.

Em seguida, usamos `format!` para adicionar o conteúdo do arquivo como corpo da resposta de sucesso. Para garantir uma resposta HTTP válida, adicionamos o cabeçalho `Content-Length`, que é definido como o tamanho do corpo da nossa resposta — neste caso, o tamanho de `hello.html`.

Execute este código com `cargo run` e carregue _127.0.0.1:7878_ em seu navegador; você deverá ver seu HTML renderizado!

Atualmente, estamos ignorando os dados da requisição em `http_request` e apenas enviando de volta o conteúdo do arquivo HTML incondicionalmente. Isso significa que, se você tentar requisitar _127.0.0.1:7878/something-else_ em seu navegador, ainda receberá esta mesma resposta HTML. No momento, nosso servidor é muito limitado e não faz o que a maioria dos servidores web faz. Queremos personalizar nossas respostas dependendo da requisição e enviar o arquivo HTML de volta apenas para uma requisição bem formada para _/_.

## Validando a Requisição e Respondendo Seletivamente

No momento, nosso servidor web retornará o HTML no arquivo não importa o que o cliente tenha requisitado. Vamos adicionar funcionalidade para verificar se o navegador está requisitando _/_ antes de retornar o arquivo HTML e para retornar um erro se o navegador requisitar qualquer outra coisa. Para isso, precisamos modificar `handle_connection`, conforme mostrado na Listagem 21-6. Este novo código verifica o conteúdo da requisição recebida em relação ao que sabemos que uma requisição para _/_ se parece e adiciona blocos `if` e `else` para tratar requisições de forma diferente.

**Arquivo: src/main.rs**

```rust
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // some other request
    }
}
```

<a id="listagem-21-6"></a>

[Listagem 21-6](#listagem-21-6): Tratando requisições para */* de forma diferente das demais requisições

Estaremos olhando apenas para a primeira linha da requisição HTTP, então, em vez de ler toda a requisição em um vetor, estamos chamando `next` para obter o primeiro item do iterador. O primeiro `unwrap` cuida do `Option` e para o programa se o iterador não tiver itens. O segundo `unwrap` trata o `Result` e tem o mesmo efeito do `unwrap` que estava no `map` adicionado na Listagem 21-2.

Em seguida, verificamos a `request_line` para ver se ela é igual à linha de requisição de uma requisição GET para o caminho _/_. Se for, o bloco `if` retorna o conteúdo do nosso arquivo HTML.

Se a `request_line` *não* for igual à requisição GET para o caminho _/_, significa que recebemos alguma outra requisição. Adicionaremos código ao bloco `else` em instantes para responder a todas as outras requisições.

Execute este código agora e requisite _127.0.0.1:7878_; você deverá obter o HTML em _hello.html_. Se fizer qualquer outra requisição, como _127.0.0.1:7878/something-else_, receberá um erro de conexão como aqueles que viu ao executar o código da Listagem 21-1 e da Listagem 21-2.

Agora vamos adicionar o código da Listagem 21-7 ao bloco `else` para retornar uma resposta com o código de status 404, que sinaliza que o conteúdo da requisição não foi encontrado. Também retornaremos algum HTML para uma página renderizar no navegador indicando a resposta ao usuário final.

**Arquivo: src/main.rs**

```rust
    // --snip--
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
```

<a id="listagem-21-7"></a>

[Listagem 21-7](#listagem-21-7): Respondendo com código de status 404 e uma página de erro se algo diferente de */* foi requisitado

Aqui, nossa resposta tem uma linha de status com código de status 404 e a frase de razão `NOT FOUND`. O corpo da resposta será o HTML no arquivo _404.html_. Você precisará criar um arquivo _404.html_ ao lado de _hello.html_ para a página de erro; novamente, sinta-se à vontade para usar qualquer HTML que quiser, ou use o HTML de exemplo da Listagem 21-8.

**Arquivo: 404.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
```

<a id="listagem-21-8"></a>

[Listagem 21-8](#listagem-21-8): Conteúdo de exemplo para a página a enviar de volta com qualquer resposta 404

Com essas alterações, execute seu servidor novamente. Requisitar _127.0.0.1:7878_ deve retornar o conteúdo de _hello.html_, e qualquer outra requisição, como _127.0.0.1:7878/foo_, deve retornar o HTML de erro de _404.html_.

<a id="a-touch-of-refactoring"></a>

## Refatoração

No momento, os blocos `if` e `else` têm muita repetição: ambos leem arquivos e escrevem o conteúdo dos arquivos no stream. As únicas diferenças são a linha de status e o nome do arquivo. Vamos tornar o código mais conciso extraindo essas diferenças para linhas `if` e `else` separadas que atribuirão os valores da linha de status e do nome do arquivo a variáveis; então podemos usar essas variáveis incondicionalmente no código para ler o arquivo e escrever a resposta. A Listagem 21-9 mostra o código resultante depois de substituir os grandes blocos `if` e `else`.

**Arquivo: src/main.rs**

```rust
use std::{
    fs,
    io::{BufReader, prelude::*},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

<a id="listagem-21-9"></a>

[Listagem 21-9](#listagem-21-9): Refatorando os blocos `if` e `else` para conter apenas o código que difere entre os dois casos

Agora os blocos `if` e `else` apenas retornam os valores apropriados para a linha de status e o nome do arquivo em uma tupla; então usamos desestruturação para atribuir esses dois valores a `status_line` e `filename` usando um padrão na instrução `let`, como discutido no Capítulo 19.

O código previamente duplicado agora está fora dos blocos `if` e `else` e usa as variáveis `status_line` e `filename`. Isso torna mais fácil ver a diferença entre os dois casos e significa que temos apenas um lugar para atualizar o código se quisermos alterar como a leitura do arquivo e a escrita da resposta funcionam. O comportamento do código da Listagem 21-9 será o mesmo da Listagem 21-7.

Incrível! Agora temos um servidor web simples em aproximadamente 40 linhas de código Rust que responde a uma requisição com uma página de conteúdo e responde a todas as outras requisições com uma resposta 404.

Atualmente, nosso servidor roda em uma única thread, o que significa que só pode atender uma requisição por vez. Vamos examinar como isso pode ser um problema simulando algumas requisições lentas. Depois, corrigiremos para que nosso servidor possa lidar com várias requisições ao mesmo tempo.
