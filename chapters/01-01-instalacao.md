---
title: "Instalação"
chapter_code: 01-01
slug: instalacao
---

# Instalação

O primeiro passo é instalar o Rust. Vamos baixar o Rust através do `rustup`, uma ferramenta de linha de comando para gerenciar versões do Rust e ferramentas associadas. Você precisará de uma conexão com a internet para o download.

> **Nota:** Se você preferir não usar o `rustup` por algum motivo, consulte a página [outros métodos de instalação do rust](https://forge.rust-lang.org/infra/other-installation-methods.html) para mais opções.

Os passos a seguir instalam a versão estável mais recente do compilador Rust. As garantias de estabilidade do Rust asseguram que todos os exemplos do livro que compilam continuarão compilando com versões mais novas do Rust. A saída pode diferir ligeiramente entre versões, porque o Rust frequentemente melhora mensagens de erro e avisos. Em outras palavras, qualquer versão estável mais recente do Rust instalada com esses passos deve funcionar como esperado com o conteúdo deste livro.

### Notação de linha de comando

Neste capítulo e ao longo do livro, mostraremos alguns comandos usados no terminal. As linhas que você deve digitar no terminal começam com `$`. Você não precisa digitar o caractere `$`; ele é apenas o prompt da linha de comando que indica o início de cada comando. Linhas que não começam com `$` normalmente mostram a saída do comando anterior. Além disso, exemplos específicos do PowerShell usarão `>` em vez de `$`.

### Instalando o `rustup` no Linux ou macOS

Se você estiver usando Linux ou macOS, abra um terminal e digite o seguinte comando:

```bash
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Esse comando baixa um script e inicia a instalação da ferramenta `rustup`, que instala a versão estável mais recente do Rust. Pode ser que você precise digitar sua senha. Se a instalação for bem-sucedida, a seguinte linha aparecerá:

```
Rust is installed now. Great!
```

Você também precisará de um *linker*, que é um programa que o Rust usa para unir suas saídas compiladas em um único arquivo. É provável que você já tenha um. Se você encontrar erros relacionados ao linker, deverá instalar um compilador C, que normalmente já inclui um linker. Um compilador C também é útil porque alguns pacotes comuns do Rust dependem de código C e precisarão dele.

No macOS, você pode obter um compilador C executando:

```bash
$ xcode-select --install
```

Usuários de Linux geralmente devem instalar o GCC ou o Clang, de acordo com a documentação da sua distribuição. Por exemplo, se você usa Ubuntu, pode instalar o pacote `build-essential`.

### Instalando o `rustup` no Windows

No Windows, acesse [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) e siga as instruções para instalar o Rust. Em algum momento da instalação, você será solicitado a instalar o Visual Studio. Ele fornece um linker e as bibliotecas nativas necessárias para compilar programas. Se precisar de mais ajuda com essa etapa, consulte [https://rust-lang.github.io/rustup/installation/windows-msvc.html](https://rust-lang.github.io/rustup/installation/windows-msvc.html).

O restante deste livro usa comandos que funcionam tanto no *cmd.exe* quanto no PowerShell. Se houver diferenças específicas, explicaremos qual usar.

### Solução de problemas

Para verificar se o Rust foi instalado corretamente, abra um terminal e digite:

```bash
$ rustc --version
```

Você deverá ver o número da versão, o hash do commit e a data do commit da versão estável mais recente lançada, no seguinte formato:

```
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Se você ver essas informações, o Rust foi instalado com sucesso! Caso contrário, verifique se o Rust está na variável de ambiente `%PATH%` do sistema.

No Windows (CMD):

```
> echo %PATH%
```

No PowerShell:

```
> echo $env:Path
```

No Linux e macOS:

```bash
$ echo $PATH
```

Se tudo estiver correto e o Rust ainda não funcionar, há vários lugares onde você pode obter ajuda. Veja como entrar em contato com outros Rustaceans (um apelido divertido que usamos) na página da comunidade [(em inglês)](https://www.rust-lang.org/community) ou [(em português)](https://universorust.com/comunidade).

### Atualização e desinstalação

Depois que o Rust estiver instalado via `rustup`, atualizar para uma nova versão lançada é fácil. No terminal, execute:

```bash
$ rustup update
```

Para desinstalar o Rust e o `rustup`, execute:

```bash
$ rustup self uninstall
```

### Lendo a documentação local

A instalação do Rust também inclui uma cópia local da documentação para que você possa lê-la offline. Execute `rustup doc` para abrir a documentação local no seu navegador.

Sempre que um tipo ou função for fornecido pela biblioteca padrão e você não tiver certeza do que ele faz ou como usá-lo, consulte a documentação da API!

### Usando editores de texto e IDEs

Este livro não faz suposições sobre quais ferramentas você usa para escrever código Rust. Praticamente qualquer editor de texto serve! No entanto, muitos editores de texto e ambientes de desenvolvimento integrados (IDEs) possuem suporte nativo ao Rust. Você pode encontrar uma lista relativamente atualizada de editores e IDEs na [página de ferramentas](https://www.rust-lang.org/tools) do site oficial do Rust.

### Trabalhando offline com este livro

Em vários exemplos, usaremos pacotes Rust além da biblioteca padrão. Para acompanhar esses exemplos, você precisará ter uma conexão com a internet ou baixar essas dependências antecipadamente. Para fazer isso, execute os seguintes comandos:

```bash
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Isso irá armazenar em cache os downloads desses pacotes, para que você não precise baixá-los novamente mais tarde. Após executar esse comando, você não precisa manter a pasta `get-dependencies`. Se tiver executado esse comando, poderá usar a flag `--offline` em todos os comandos `cargo` no restante do livro, utilizando essas versões em cache em vez de acessar a rede.