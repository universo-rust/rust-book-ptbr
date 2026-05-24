---
title: "Instalando binários com `cargo install`"
chapter_code: 14-04
slug: instalando-binarios-com-cargo-install
---

# Instalando Binários com `cargo install`

O comando `cargo install` permite instalar e usar crates binários localmente. Isso não pretende substituir pacotes do sistema; é uma forma conveniente para desenvolvedores Rust instalarem ferramentas que outras pessoas compartilharam em [crates.io](https://crates.io/). Note que você só pode instalar pacotes que tenham targets binários. Um _target binário_ é o programa executável criado se o crate tiver um arquivo _src/main.rs_ ou outro arquivo especificado como binário, em oposição a um target de biblioteca que não é executável por si só, mas é adequado para incluir em outros programas. Em geral, crates têm informação no arquivo README sobre se o crate é uma biblioteca, tem um target binário ou ambos.

Todos os binários instalados com `cargo install` são armazenados na pasta _bin_ da raiz de instalação. Se você instalou o Rust usando _rustup.rs_ e não tem configurações personalizadas, este diretório será *$HOME/.cargo/bin*. Certifique-se de que este diretório está no seu `$PATH` para poder executar programas que instalou com `cargo install`.

Por exemplo, no Capítulo 12 mencionamos que existe uma implementação em Rust da ferramenta `grep` chamada `ripgrep` para buscar em arquivos. Para instalar `ripgrep`, podemos executar o seguinte:

```bash
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

A penúltima linha da saída mostra o local e o nome do binário instalado, que no caso de `ripgrep` é `rg`. Enquanto o diretório de instalação estiver no seu `$PATH`, como mencionado anteriormente, você pode executar `rg --help` e começar a usar uma ferramenta mais rápida e mais “Rusty” para buscar em arquivos!
