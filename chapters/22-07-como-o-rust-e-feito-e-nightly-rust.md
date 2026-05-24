---
title: "Como o Rust é feito e \"Nightly Rust\""
chapter_code: 22-07
slug: como-o-rust-e-feito-e-nightly-rust
---

# Apêndice G: Como o Rust é feito e “Nightly Rust”

Este apêndice trata de como o Rust é feito e de como isso afeta você como desenvolvedor Rust.

### Estabilidade sem estagnação

Como linguagem, o Rust se importa _muito_ com a estabilidade do seu código. Queremos que o Rust seja uma base sólida sobre a qual você possa construir, e se as coisas estivessem mudando constantemente, isso seria impossível. Ao mesmo tempo, se não pudermos experimentar novos recursos, podemos não descobrir falhas importantes até depois do lançamento, quando já não podemos mais mudar as coisas.

Nossa solução para este problema é o que chamamos de “estabilidade sem estagnação”, e nosso princípio orientador é este: você nunca deve ter medo de atualizar para uma nova versão estável do Rust. Cada atualização deve ser indolor, mas também deve trazer novos recursos, menos bugs e tempos de compilação mais rápidos.

### Tchau, tchau! Canais de lançamento e pegando os trens

O desenvolvimento do Rust opera em um _cronograma de trens_. Ou seja, todo o desenvolvimento é feito no branch principal do repositório do Rust. Os lançamentos seguem um modelo de release train de software, que tem sido usado pelo Cisco IOS e outros projetos de software. Existem três _canais de lançamento_ para o Rust:

- Nightly
- Beta
- Stable

A maioria dos desenvolvedores Rust usa principalmente o canal stable, mas aqueles que querem experimentar novos recursos experimentais podem usar nightly ou beta.

Aqui está um exemplo de como funciona o processo de desenvolvimento e lançamento: vamos supor que a equipe do Rust esteja trabalhando no lançamento do Rust 1.5. Esse lançamento aconteceu em dezembro de 2015, mas nos dará números de versão realistas. Um novo recurso é adicionado ao Rust: um novo commit chega no branch principal. A cada noite, uma nova versão nightly do Rust é produzida. Todo dia é um dia de lançamento, e esses lançamentos são criados pela nossa infraestrutura de release automaticamente. Então, com o passar do tempo, nossos lançamentos ficam assim, uma vez por noite:

```text
nightly: * - - * - - *
```

A cada seis semanas, é hora de preparar um novo lançamento! O branch `beta` do repositório Rust se ramifica do branch principal usado pelo nightly. Agora, há dois lançamentos:

```text
nightly: * - - * - - *
                     |
beta:                *
```

A maioria dos usuários do Rust não usa ativamente lançamentos beta, mas testa contra beta em seu sistema de CI para ajudar o Rust a descobrir possíveis regressões. Enquanto isso, ainda há um lançamento nightly toda noite:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Digamos que uma regressão seja encontrada. Ainda bem que tivemos algum tempo para testar o lançamento beta antes que a regressão entrasse em um lançamento stable! A correção é aplicada no branch principal, para que o nightly seja corrigido, e então a correção é backportada para o branch `beta`, e um novo lançamento de beta é produzido:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Seis semanas depois que o primeiro beta foi criado, é hora de um lançamento stable! O branch `stable` é produzido a partir do branch `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Viva! Rust 1.5 está pronto! No entanto, esquecemos uma coisa: como as seis semanas passaram, também precisamos de um novo beta da _próxima_ versão do Rust, 1.6. Então, depois que `stable` se ramifica de `beta`, a próxima versão de `beta` se ramifica de `nightly` novamente:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Isso é chamado de “modelo de trem” porque a cada seis semanas um lançamento “sai da estação”, mas ainda precisa fazer uma jornada pelo canal beta antes de chegar como lançamento stable.

O Rust é lançado a cada seis semanas, como um relógio. Se você souber a data de um lançamento do Rust, pode saber a data do próximo: é seis semanas depois. Um aspecto agradável de ter lançamentos programados a cada seis semanas é que o próximo trem chega em breve. Se um recurso acontecer de perder um lançamento específico, não há necessidade de se preocupar: outro está chegando em pouco tempo! Isso ajuda a reduzir a pressão para incluir recursos possivelmente não polidos perto do prazo de lançamento.

Graças a este processo, você sempre pode conferir a próxima build do Rust e verificar por si mesmo que é fácil atualizar: se um lançamento beta não funcionar como esperado, você pode reportar à equipe e fazer com que seja corrigido antes que o próximo lançamento stable aconteça! Quebras em um lançamento beta são relativamente raras, mas `rustc` ainda é um software, e bugs existem.

### Tempo de manutenção

O projeto Rust suporta a versão stable mais recente. Quando uma nova versão stable é lançada, a versão antiga atinge o fim de vida (EOL). Isso significa que cada versão é suportada por seis semanas.

### Recursos instáveis

Há mais um detalhe com este modelo de lançamento: recursos instáveis. O Rust usa uma técnica chamada “feature flags” para determinar quais recursos estão habilitados em um determinado lançamento. Se um novo recurso está em desenvolvimento ativo, ele chega no branch principal e, portanto, no nightly, mas por trás de uma _feature flag_. Se você, como usuário, deseja experimentar o recurso em andamento, pode, mas deve estar usando um lançamento nightly do Rust e anotar seu código-fonte com a flag apropriada para optar por participar.

Se você estiver usando um lançamento beta ou stable do Rust, não pode usar nenhuma feature flag. Esta é a chave que nos permite obter uso prático com novos recursos antes de declará-los estáveis para sempre. Aqueles que desejam optar pela vanguarda podem fazê-lo, e aqueles que querem uma experiência sólida podem ficar com stable e saber que seu código não quebrará. Estabilidade sem estagnação.

Este livro contém apenas informações sobre recursos estáveis, pois recursos em andamento ainda estão mudando, e certamente serão diferentes entre quando este livro foi escrito e quando forem habilitados em builds stable. Você pode encontrar documentação para recursos apenas nightly online.

### Rustup e o papel do Rust Nightly

O rustup facilita alternar entre diferentes canais de lançamento do Rust, em nível global ou por projeto. Por padrão, você terá o Rust stable instalado. Para instalar o nightly, por exemplo:

```console
$ rustup toolchain install nightly
```

Você pode ver todas as _toolchains_ (lançamentos do Rust e componentes associados) que tem instaladas com `rustup` também. Aqui está um exemplo no computador Windows de um dos autores:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Como você pode ver, a toolchain stable é a padrão. A maioria dos usuários do Rust usa stable na maior parte do tempo. Você pode querer usar stable na maior parte do tempo, mas usar nightly em um projeto específico, porque se importa com um recurso de ponta. Para fazer isso, você pode usar `rustup override` no diretório desse projeto para definir a toolchain nightly como aquela que o `rustup` deve usar quando você estiver nesse diretório:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Agora, sempre que você chamar `rustc` ou `cargo` dentro de _~/projects/needs-nightly_, o `rustup` garantirá que você está usando o Rust nightly, em vez do seu padrão stable. Isso é útil quando você tem muitos projetos Rust!

### O processo RFC e as equipes

Então, como você aprende sobre esses novos recursos? O modelo de desenvolvimento do Rust segue um processo de _Request For Comments (RFC)_. Se você quiser uma melhoria no Rust, pode escrever uma proposta, chamada RFC.

Qualquer pessoa pode escrever RFCs para melhorar o Rust, e as propostas são revisadas e discutidas pela equipe do Rust, que é composta por muitas subequipes temáticas. Há uma lista completa das equipes [no site do Rust](https://www.rust-lang.org/governance), que inclui equipes para cada área do projeto: design da linguagem, implementação do compilador, infraestrutura, documentação e mais. A equipe apropriada lê a proposta e os comentários, escreve alguns comentários próprios e, eventualmente, há consenso para aceitar ou rejeitar o recurso.

Se o recurso for aceito, uma issue é aberta no repositório do Rust, e alguém pode implementá-lo. A pessoa que implementa muito bem pode não ser a pessoa que propôs o recurso em primeiro lugar! Quando a implementação estiver pronta, ela chega no branch principal por trás de uma feature gate, como discutimos na seção [“Recursos instáveis”](#recursos-instáveis).

Depois de algum tempo, assim que desenvolvedores Rust que usam lançamentos nightly tiverem podido experimentar o novo recurso, membros da equipe discutirão o recurso, como funcionou no nightly, e decidirão se ele deve entrar no Rust stable ou não. Se a decisão for seguir em frente, a feature gate é removida e o recurso passa a ser considerado estável! Ele pega os trens até um novo lançamento stable do Rust.
