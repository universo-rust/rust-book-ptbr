---
title: "Estendendo o Cargo"
chapter_code: 14-05
slug: estendendo-o-cargo
---

# Estendendo o Cargo com Comandos Personalizados

O Cargo foi projetado para que você possa estendê-lo com novos subcomandos sem precisar modificá-lo. Se um binário no seu `$PATH` se chama `cargo-something`, você pode executá-lo como se fosse um subcomando do Cargo executando `cargo something`. Comandos personalizados assim também aparecem listados quando você executa `cargo --list`. Poder usar `cargo install` para instalar extensões e depois executá-las como as ferramentas integradas do Cargo é um benefício super conveniente do design do Cargo!

## Resumo

Compartilhar código com o Cargo e [crates.io](https://crates.io/) faz parte do que torna o ecossistema Rust útil para muitas tarefas diferentes. A biblioteca padrão do Rust é pequena e estável, mas crates são fáceis de compartilhar, usar e melhorar em um cronograma diferente do da linguagem. Não tenha vergonha de compartilhar em [crates.io](https://crates.io/) código que é útil para você; é provável que também seja útil para outra pessoa!
