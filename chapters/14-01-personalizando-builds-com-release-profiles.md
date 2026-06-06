---
title: "Personalizando builds com release profiles"
chapter_code: 14-01
slug: personalizando-builds-com-release-profiles
challenge_day: 18
reading_minutes: 2
---

# Personalizando builds com release profiles

Em Rust, _release profiles_ são perfis predefinidos e personalizáveis com configurações diferentes que permitem ao programador ter mais controle sobre várias opções de compilação de código. Cada perfil é configurado independentemente dos outros.

O Cargo tem dois perfis principais: o perfil `dev` que o Cargo usa quando você executa `cargo build`, e o perfil `release` que o Cargo usa quando você executa `cargo build --release`. O perfil `dev` é definido com bons padrões para desenvolvimento, e o perfil `release` tem bons padrões para builds de release.

Esses nomes de perfil podem ser familiares pela saída das suas compilações:

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` e `release` são esses perfis diferentes usados pelo compilador.

O Cargo tem configurações padrão para cada um dos perfis que se aplicam quando você não adicionou explicitamente nenhuma seção `[profile.*]` no arquivo _Cargo.toml_ do projeto. Ao adicionar seções `[profile.*]` para qualquer perfil que queira personalizar, você substitui qualquer subconjunto das configurações padrão. Por exemplo, aqui estão os valores padrão para a configuração `opt-level` dos perfis `dev` e `release`:

**Arquivo: Cargo.toml**

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

A configuração `opt-level` controla o número de otimizações que o Rust aplicará ao seu código, com intervalo de 0 a 3. Aplicar mais otimizações aumenta o tempo de compilação; se você está em desenvolvimento e compila seu código com frequência, vai querer menos otimizações para compilar mais rápido, mesmo que o código resultante execute mais devagar. O `opt-level` padrão para `dev` é portanto `0`. Quando estiver pronto para lançar seu código, é melhor gastar mais tempo compilando. Você só compilará em modo release uma vez, mas executará o programa compilado muitas vezes; o modo release troca tempo de compilação maior por código que executa mais rápido. Por isso o `opt-level` padrão do perfil `release` é `3`.

Você pode substituir uma configuração padrão adicionando um valor diferente para ela no _Cargo.toml_. Por exemplo, se quisermos usar nível de otimização 1 no perfil de desenvolvimento, podemos adicionar estas duas linhas ao _Cargo.toml_ do projeto:

**Arquivo: Cargo.toml**

```toml
[profile.dev]
opt-level = 1
```

Este código substitui a configuração padrão de `0`. Agora, quando executarmos `cargo build`, o Cargo usará os padrões do perfil `dev` mais nossa personalização de `opt-level`. Como definimos `opt-level` como `1`, o Cargo aplicará mais otimizações do que o padrão, mas não tantas quanto em um build de release.

Para a lista completa de opções de configuração e padrões de cada perfil, consulte a [documentação do Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
